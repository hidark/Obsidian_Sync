# AI多角色实时语音对话节目 - 软件架构设计

## 1. 整体架构

### 1.1 分层架构

```
┌─────────────────────────────────────────────────────────────┐
│                    用户界面层 (UI Layer)                      │
│  - Web前端 (React/Vue)                                       │
│  - 实时音频采集与播放                                         │
│  - 角色选择与对话控制面板                                     │
└─────────────────────────────────────────────────────────────┘
                            ↕ WebSocket
┌─────────────────────────────────────────────────────────────┐
│                  业务逻辑层 (Service Layer)                   │
│  - 对话管理器 (DialogueManager)                              │
│  - 角色编排器 (CharacterOrchestrator)                        │
│  - 上下文管理器 (ContextManager)                             │
│  - 流式处理协调器 (StreamCoordinator)                        │
└─────────────────────────────────────────────────────────────┘
                            ↕
┌─────────────────────────────────────────────────────────────┐
│                   AI服务层 (AI Service Layer)                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ STT Service  │  │  LLM Service │  │ TTS Service  │      │
│  │   (Whisper)  │  │   (Claude)   │  │   (Coqui)    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            ↕
┌─────────────────────────────────────────────────────────────┐
│                基础设施层 (Infrastructure Layer)              │
│  - Redis (上下文缓存、预生成队列)                            │
│  - SQLite/PostgreSQL (对话历史、角色配置)                    │
│  - 文件存储 (音频文件、模型文件)                             │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 技术栈选型

**核心技术栈：Python + FastAPI**

**理由**：
- Python生态对AI模型支持最好（Whisper、Coqui TTS原生支持）
- FastAPI提供高性能异步处理和WebSocket支持
- 便于集成科学计算库（numpy、torch）

**完整技术栈**：

| 层级 | 技术 | 理由 |
|------|------|------|
| 前端 | React + TypeScript | 组件化UI、WebSocket原生支持、Web Audio API |
| 后端 | Python 3.11 + FastAPI | 异步IO、AI库生态、WebSocket支持 |
| 实时通信 | WebSocket | 双向低延迟通信，优于SSE（需要双向音频流） |
| STT | faster-whisper (CTranslate2) | 比原版Whisper快4倍，内存占用减半 |
| LLM | Claude API (claude-sonnet-4-20250514) | 性价比最优，支持Prompt Caching |
| TTS | Coqui XTTS-v2 | 开源免费、多音色克隆、中文支持 |
| 缓存 | Redis | 上下文缓存、预生成结果队列、发布订阅 |
| 数据库 | SQLite | 轻量级，本地部署无需额外服务 |
| 任务队列 | asyncio + Redis Pub/Sub | 预生成任务调度 |
| 容器化 | Docker + docker-compose | 一键部署，环境隔离 |

---

## 2. 核心模块设计

### 2.1 系统模块关系图

```
                    ┌──────────────┐
                    │   Web 前端    │
                    │  (React App) │
                    └──────┬───────┘
                           │ WebSocket (音频流 + JSON控制消息)
                           ▼
                    ┌──────────────┐
                    │  WebSocket   │
                    │   Gateway    │
                    └──────┬───────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
     ┌────────────┐ ┌───────────┐ ┌──────────┐
     │ STT Module │ │ Dialogue  │ │   TTS    │
     │  (Whisper) │ │  Manager  │ │  Module  │
     └─────┬──────┘ └─────┬─────┘ └────┬─────┘
           │               │             │
           │        ┌──────┴──────┐      │
           │        ▼             ▼      │
           │  ┌──────────┐ ┌─────────┐  │
           └─→│ Context  │ │Character│←─┘
              │ Manager  │ │ Engine  │
              └────┬─────┘ └────┬────┘
                   │            │
                   ▼            ▼
              ┌─────────┐ ┌─────────┐
              │  Redis  │ │ Claude  │
              │ (Cache) │ │   API   │
              └─────────┘ └─────────┘
```

### 2.2 语音输入模块 (STT Module)

**选型：faster-whisper（基于CTranslate2优化的Whisper）**

关键设计决策：
- 使用 `large-v3` 模型保证中文识别准确率
- VAD（语音活动检测）前置过滤静音段，减少无效推理
- 流式分块处理：前端每300ms发送一个音频块，后端累积到有效语音段后触发识别

```python
# stt/whisper_service.py

import numpy as np
from faster_whisper import WhisperModel
from dataclasses import dataclass
import asyncio

@dataclass
class STTConfig:
    model_size: str = "large-v3"
    device: str = "cuda"           # GPU推理
    compute_type: str = "float16"  # 半精度，速度翻倍
    language: str = "zh"
    vad_threshold: float = 0.5
    min_speech_duration: float = 0.5  # 最短语音段（秒）

class WhisperSTTService:
    def __init__(self, config: STTConfig):
        self.config = config
        self.model = WhisperModel(
            config.model_size,
            device=config.device,
            compute_type=config.compute_type,
        )
        self._audio_buffer = bytearray()
        self._sample_rate = 16000

    async def transcribe_stream(self, audio_chunk: bytes) -> str | None:
        """
        接收音频块，累积后进行识别。
        返回识别文本或None（未达到有效语音段）。
        """
        self._audio_buffer.extend(audio_chunk)

        # 累积至少1秒音频再处理
        min_bytes = self._sample_rate * 2  # 16-bit PCM = 2 bytes/sample
        if len(self._audio_buffer) < min_bytes:
            return None

        audio_np = np.frombuffer(
            bytes(self._audio_buffer), dtype=np.int16
        ).astype(np.float32) / 32768.0

        # 在线程池中运行CPU/GPU密集任务，避免阻塞事件循环
        loop = asyncio.get_event_loop()
        segments, info = await loop.run_in_executor(
            None,
            lambda: self.model.transcribe(
                audio_np,
                language=self.config.language,
                vad_filter=True,
                vad_parameters={"threshold": self.config.vad_threshold},
            ),
        )

        text = "".join(seg.text for seg in segments).strip()
        self._audio_buffer.clear()

        return text if text else None

    def reset(self):
        """重置音频缓冲区（新一轮对话时调用）"""
        self._audio_buffer.clear()
```

**技术难点与风险**：
- [风险] Whisper large-v3 需要约6GB显存，与TTS共享GPU时需注意显存分配
- [难点] 实时VAD的灵敏度调参：太灵敏会把背景噪音当语音，太迟钝会截断用户发言
- [缓解] 可降级到 `medium` 模型（约2GB显存），中文准确率仍可接受

---

### 2.3 对话管理器 (Dialogue Manager)

**核心职责**：
1. 维护对话状态机（等待用户输入 → 角色发言中 → 预生成其他角色反应）
2. 协调各模块的调用顺序
3. 管理发言队列（用户指定角色 → 播放预生成内容或实时生成）

```python
# dialogue/manager.py

from enum import Enum
from typing import Dict, List, Optional
from dataclasses import dataclass, field
import asyncio

class DialogueState(Enum):
    IDLE = "idle"                    # 等待用户输入
    LISTENING = "listening"          # 正在录音
    PROCESSING_STT = "processing_stt"  # 语音识别中
    CHARACTER_SPEAKING = "character_speaking"  # 角色发言中
    PRE_GENERATING = "pre_generating"  # 后台预生成其他角色反应

@dataclass
class Turn:
    """单轮对话"""
    speaker: str  # "user" 或角色名
    text: str
    audio_path: Optional[str] = None  # TTS生成的音频文件路径
    timestamp: float = field(default_factory=lambda: time.time())

@dataclass
class PreGeneratedResponse:
    """预生成的角色反应"""
    character_id: str
    text: str
    audio_path: Optional[str] = None
    context_hash: str  # 基于上下文的哈希，用于判断是否过期

class DialogueManager:
    def __init__(
        self,
        context_manager,
        character_engine,
        tts_service,
        redis_client,
    ):
        self.context = context_manager
        self.characters = character_engine
        self.tts = tts_service
        self.redis = redis_client

        self.state = DialogueState.IDLE
        self.history: List[Turn] = []
        self.pre_generated: Dict[str, PreGeneratedResponse] = {}

    async def handle_user_input(
        self, text: str, target_character_id: str
    ) -> Turn:
        """
        处理用户输入，让指定角色回答。
        流程：
        1. 记录用户发言
        2. 生成目标角色回答（流式）
        3. 触发其他角色预生成反应
        """
        self.state = DialogueState.CHARACTER_SPEAKING

        # 1. 记录用户发言
        user_turn = Turn(speaker="user", text=text)
        self.history.append(user_turn)
        await self.context.add_turn(user_turn)

        # 2. 生成目标角色回答（流式）
        character_response = ""
        async for chunk in self.characters.generate_response_stream(
            character_id=target_character_id,
            context=await self.context.get_context(),
        ):
            character_response += chunk
            # 实时发送给前端显示
            yield {"type": "text_chunk", "text": chunk}

        # 3. 生成TTS音频
        audio_path = await self.tts.synthesize(
            text=character_response,
            voice_id=self.characters.get_voice_id(target_character_id),
        )

        character_turn = Turn(
            speaker=target_character_id,
            text=character_response,
            audio_path=audio_path,
        )
        self.history.append(character_turn)
        await self.context.add_turn(character_turn)

        # 4. 触发其他角色预生成（异步后台任务）
        asyncio.create_task(
            self._pre_generate_reactions(
                responding_character=target_character_id
            )
        )

        self.state = DialogueState.IDLE
        return character_turn

    async def get_character_response(
        self, character_id: str, user_followup: Optional[str] = None
    ) -> Turn:
        """
        获取指定角色的回应。
        - 如果有预生成内容且上下文未变，直接返回
        - 否则实时生成
        """
        context_hash = await self.context.get_hash()

        # 检查预生成缓存
        if character_id in self.pre_generated:
            cached = self.pre_generated[character_id]
            if cached.context_hash == context_hash and not user_followup:
                # 缓存有效，直接返回
                turn = Turn(
                    speaker=character_id,
                    text=cached.text,
                    audio_path=cached.audio_path,
                )
                self.history.append(turn)
                await self.context.add_turn(turn)
                return turn

        # 缓存失效或有追问，重新生成
        if user_followup:
            # 用户追问，需要结合追问内容
            user_turn = Turn(speaker="user", text=user_followup)
            self.history.append(user_turn)
            await self.context.add_turn(user_turn)

        # 实时生成
        response_text = await self.characters.generate_response(
            character_id=character_id,
            context=await self.context.get_context(),
        )

        audio_path = await self.tts.synthesize(
            text=response_text,
            voice_id=self.characters.get_voice_id(character_id),
        )

        turn = Turn(
            speaker=character_id, text=response_text, audio_path=audio_path
        )
        self.history.append(turn)
        await self.context.add_turn(turn)

        return turn

    async def _pre_generate_reactions(self, responding_character: str):
        """
        后台预生成其他角色对当前发言的反应。
        """
        self.state = DialogueState.PRE_GENERATING
        context = await self.context.get_context()
        context_hash = await self.context.get_hash()

        other_characters = [
            c for c in self.characters.get_all_ids()
            if c != responding_character
        ]

        tasks = []
        for char_id in other_characters:
            tasks.append(
                self._generate_and_cache_reaction(
                    char_id, context, context_hash
                )
            )

        await asyncio.gather(*tasks)
        self.state = DialogueState.IDLE

    async def _generate_and_cache_reaction(
        self, character_id: str, context: str, context_hash: str
    ):
        """生成单个角色的反应并缓存"""
        text = await self.characters.generate_response(
            character_id=character_id, context=context
        )

        audio_path = await self.tts.synthesize(
            text=text,
            voice_id=self.characters.get_voice_id(character_id),
        )

        self.pre_generated[character_id] = PreGeneratedResponse(
            character_id=character_id,
            text=text,
            audio_path=audio_path,
            context_hash=context_hash,
        )

        # 同时存入Redis，支持分布式部署
        await self.redis.setex(
            f"pre_gen:{character_id}:{context_hash}",
            300,  # 5分钟过期
            json.dumps({
                "text": text,
                "audio_path": audio_path,
            }),
        )
```

**关键设计点**：
- 预生成策略：角色A发言后，立即预生成B、C、D的反应，用户点击时秒播
- 上下文哈希：用于判断预生成内容是否过期（如果用户追问，上下文变化，缓存失效）
- 异步任务：预生成不阻塞主流程，用户体验流畅

---

### 2.4 AI角色引擎 (Character Engine)

**核心职责**：
1. 管理各角色的System Prompt
2. 调用Claude API，利用Prompt Caching降低成本
3. 支持流式输出

```python
# character/engine.py

import anthropic
import hashlib
from typing import AsyncGenerator, Dict, List
from dataclasses import dataclass

@dataclass
class CharacterConfig:
    id: str
    name: str
    voice_id: str  # Coqui TTS音色标识
    system_prompt: str
    personality_traits: List[str]
    debate_style: str  # 辩论风格描述

# 角色配置示例
CHARACTERS = {
    "tech_optimist": CharacterConfig(
        id="tech_optimist",
        name="技术乐观派·李明",
        voice_id="male_energetic",
        system_prompt="""你是"技术乐观派·李明"，一位资深科技创业者和技术布道者。

核心立场：
- 坚信技术进步是解决人类问题的根本途径
- 对AI、量子计算、生物技术等前沿领域充满热情
- 认为技术带来的短期阵痛是值得的，长期收益远大于风险

辩论风格：
- 用具体的技术案例和数据支撑观点
- 善于类比历史上的技术革命（印刷术、蒸汽机、互联网）
- 语气热情但不失理性，偶尔带点极客幽默
- 回应批评时承认问题但强调解决方案

语言要求：
- 使用中文回答
- 每次发言控制在100-200字
- 口语化表达，像在录播客""",
        personality_traits=["热情", "数据驱动", "前瞻性"],
        debate_style="用案例和数据说话，承认问题但聚焦解决方案",
    ),
    "business_realist": CharacterConfig(
        id="business_realist",
        name="商业现实派·王芳",
        voice_id="female_professional",
        system_prompt="""你是"商业现实派·王芳"，一位经验丰富的投资人和商业顾问。

核心立场：
- 关注技术的商业可行性和投资回报
- 重视市场验证、用户需求和盈利模式
- 对"技术万能论"持怀疑态度，强调执行力和商业模式

辩论风格：
- 用商业案例和财务数据说话
- 善于指出技术方案的商业盲点
- 语气冷静专业，偶尔犀利
- 不否定技术价值，但强调"好技术≠好生意"

语言要求：
- 使用中文回答
- 每次发言控制在100-200字
- 口语化表达，像在录播客""",
        personality_traits=["务实", "犀利", "数据敏感"],
        debate_style="从商业角度切入，用ROI和市场数据反驳",
    ),
    "humanist_critic": CharacterConfig(
        id="humanist_critic",
        name="人文批判派·张教授",
        voice_id="male_scholarly",
        system_prompt="""你是"人文批判派·张教授"，一位哲学系教授和科技伦理研究者。

核心立场：
- 关注技术对人类社会、文化、伦理的深层影响
- 警惕技术异化、数字鸿沟、隐私侵蚀等问题
- 主张技术发展需要人文价值观的引导

辩论风格：
- 引用哲学家、社会学家的观点
- 善于提出深层次的反思性问题
- 语气温和但观点尖锐
- 不反对技术本身，反对不加反思的技术崇拜

语言要求：
- 使用中文回答
- 每次发言控制在100-200字
- 口语化表达，像在录播客""",
        personality_traits=["深思", "温和", "批判性"],
        debate_style="从人文视角提出反思，引用经典理论",
    ),
}


class CharacterEngine:
    def __init__(self, characters: Dict[str, CharacterConfig] = None):
        self.client = anthropic.AsyncAnthropic()
        self.characters = characters or CHARACTERS
        self._prompt_cache: Dict[str, str] = {}

    def get_all_ids(self) -> List[str]:
        return list(self.characters.keys())

    def get_voice_id(self, character_id: str) -> str:
        return self.characters[character_id].voice_id

    async def generate_response_stream(
        self, character_id: str, context: str
    ) -> AsyncGenerator[str, None]:
        """
        流式生成角色回答。
        利用Prompt Caching：system prompt标记为cache_control，
        多次调用同一角色时命中缓存。
        """
        character = self.characters[character_id]

        async with self.client.messages.stream(
            model="claude-sonnet-4-20250514",
            max_tokens=500,
            system=[
                {
                    "type": "text",
                    "text": character.system_prompt,
                    "cache_control": {"type": "ephemeral"},
                    # ↑ 关键：标记system prompt为可缓存
                    # 同一角色的后续调用将命中缓存
                    # 缓存命中时输入token成本降低90%
                }
            ],
            messages=[
                {
                    "role": "user",
                    "content": context,
                }
            ],
        ) as stream:
            async for text in stream.text_stream:
                yield text

    async def generate_response(
        self, character_id: str, context: str
    ) -> str:
        """非流式生成（用于预生成场景）"""
        chunks = []
        async for chunk in self.generate_response_stream(
            character_id, context
        ):
            chunks.append(chunk)
        return "".join(chunks)
```

**Prompt Caching 成本优化分析**：

```
角色System Prompt平均长度：~400 tokens
对话上下文平均长度：~1000 tokens

无缓存：每次调用 = 400 + 1000 = 1400 input tokens
有缓存：首次调用 = 1400 tokens（写入缓存）
        后续调用 = 400 cached + 1000 new = 1000 new tokens
        缓存命中的400 tokens按0.1倍计费

一场30分钟节目约50次API调用：
  无缓存：50 × 1400 = 70,000 input tokens ≈ $0.21
  有缓存：1400 + 49 × (40 + 1000) = 52,360 等效tokens ≈ $0.16
  节省约 24%
```

**技术难点**：
- [难点] 上下文长度控制：对话历史累积后可能超过模型上下文窗口（200K tokens）
- [缓解] 实现滑动窗口机制，保留最近10轮对话 + 关键摘要
- [风险] 角色"串戏"：同一模型模拟多角色，可能出现角色混淆
- [缓解] System Prompt中强化角色身份，在上下文中明确标注发言者

---

### 2.5 语音输出模块 (TTS Module)

**选型：Coqui XTTS-v2**

关键优势：
- 支持音色克隆（只需10秒音频样本）
- 中文支持良好
- 可本地部署，无API成本

```python
# tts/coqui_service.py

from TTS.api import TTS
import torch
import asyncio
import hashlib
from pathlib import Path

class CoquiTTSService:
    def __init__(self, model_name: str = "tts_models/multilingual/multi-dataset/xtts_v2"):
        self.device = "cuda" if torch.cuda.is_available() else "cpu"
        self.tts = TTS(model_name).to(self.device)

        # 预加载角色音色样本
        self.voice_samples = {
            "male_energetic": "voices/male_energetic.wav",
            "female_professional": "voices/female_professional.wav",
            "male_scholarly": "voices/male_scholarly.wav",
        }

        self.cache_dir = Path("audio_cache")
        self.cache_dir.mkdir(exist_ok=True)

    async def synthesize(self, text: str, voice_id: str) -> str:
        """
        合成语音，返回音频文件路径。
        使用文本哈希作为缓存键，避免重复合成。
        """
        # 缓存检查
        text_hash = hashlib.md5(
            f"{text}:{voice_id}".encode()
        ).hexdigest()
        cache_path = self.cache_dir / f"{text_hash}.wav"

        if cache_path.exists():
            return str(cache_path)

        # 在线程池中运行TTS（CPU/GPU密集）
        loop = asyncio.get_event_loop()
        await loop.run_in_executor(
            None,
            lambda: self.tts.tts_to_file(
                text=text,
                speaker_wav=self.voice_samples[voice_id],
                language="zh-cn",
                file_path=str(cache_path),
            ),
        )

        return str(cache_path)

    async def synthesize_stream(
        self, text: str, voice_id: str
    ) -> AsyncGenerator[bytes, None]:
        """
        流式TTS（实验性）：边生成边播放。
        需要将文本分句，逐句合成。
        """
        sentences = self._split_sentences(text)

        for sentence in sentences:
            audio_path = await self.synthesize(sentence, voice_id)
            with open(audio_path, "rb") as f:
                audio_data = f.read()
            yield audio_data

    def _split_sentences(self, text: str) -> List[str]:
        """按标点符号分句"""
        import re
        sentences = re.split(r'[。！？；]', text)
        return [s.strip() for s in sentences if s.strip()]
```

**性能优化**：
- 音频缓存：相同文本+音色的TTS结果缓存，避免重复合成
- 预热策略：系统启动时预生成常用短语（"好的"、"我认为"等）
- 分句流式：长文本分句合成，边合成边播放，降低首字延迟

**技术难点**：
- [难点] TTS延迟：XTTS-v2合成100字约需2-3秒
- [缓解] 预生成策略 + 音频缓存，用户点击时直接播放
- [风险] 音色一致性：克隆音色需要高质量样本
- [缓解] 提供默认音色库，或让用户上传专业录音

---

### 2.6 上下文管理器 (Context Manager)

**核心职责**：
1. 维护对话历史
2. 生成发送给Claude的上下文字符串
3. 实现滑动窗口，控制上下文长度

```python
# context/manager.py

import hashlib
import json
from typing import List
from dataclasses import asdict

class ContextManager:
    def __init__(self, max_turns: int = 10):
        self.max_turns = max_turns
        self.turns: List[Turn] = []

    async def add_turn(self, turn: Turn):
        """添加一轮对话"""
        self.turns.append(turn)

        # 滑动窗口：保留最近N轮
        if len(self.turns) > self.max_turns:
            self.turns = self.turns[-self.max_turns:]

    async def get_context(self) -> str:
        """
        生成发送给Claude的上下文字符串。
        格式：
        [对话历史]
        用户: xxx
        技术乐观派·李明: xxx
        商业现实派·王芳: xxx
        ...
        """
        context_lines = ["[对话历史]"]

        for turn in self.turns:
            speaker = "用户" if turn.speaker == "user" else turn.speaker
            context_lines.append(f"{speaker}: {turn.text}")

        return "\n".join(context_lines)

    async def get_hash(self) -> str:
        """
        生成上下文哈希，用于判断预生成内容是否过期。
        """
        context_str = await self.get_context()
        return hashlib.md5(context_str.encode()).hexdigest()

    async def get_summary(self) -> str:
        """
        生成对话摘要（用于超长对话场景）。
        可调用Claude API生成摘要，替换旧对话。
        """
        # TODO: 实现摘要逻辑
        pass
```

---

## 3. 数据流与时序

### 3.1 完整请求-响应流程

```
用户发言 → 指定角色A回答 → 其他角色预生成反应 → 用户指定角色B → 播放预生成内容

详细步骤：

1. 用户按住"说话"按钮
   ├─ 前端：开始录音，WebSocket发送音频流
   └─ 后端：STT模块接收音频块

2. 用户松开按钮
   ├─ 前端：停止录音，发送 {type: "end_speech"}
   └─ 后端：STT完成识别，返回文本

3. 用户在UI选择"让技术乐观派回答"
   ├─ 前端：发送 {type: "request_response", character: "tech_optimist"}
   └─ 后端：DialogueManager.handle_user_input()

4. 后端生成角色回答
   ├─ CharacterEngine调用Claude API（流式）
   ├─ 实时返回文本块给前端显示
   └─ 完整文本生成后，调用TTS合成音频

5. 后端返回音频
   ├─ 发送 {type: "audio_ready", url: "/audio/xxx.wav"}
   └─ 前端：播放音频

6. 后台预生成其他角色反应（异步）
   ├─ 并行调用Claude API生成"商业现实派"和"人文批判派"的回应
   ├─ 并行调用TTS合成音频
   └─ 缓存到Redis和内存

7. 用户点击"听听商业现实派怎么说"
   ├─ 前端：发送 {type: "request_response", character: "business_realist"}
   ├─ 后端：检查预生成缓存，命中
   └─ 直接返回音频URL，秒播
```

### 3.2 时序图（从用户说话到AI语音播放）

```
用户      前端        WebSocket      STT        Dialogue    Character    TTS       Claude
 │         │            │            │          Manager      Engine      │         API
 │         │            │            │            │            │          │          │
 │ 按住按钮 │            │            │            │            │          │          │
 ├────────>│            │            │            │            │          │          │
 │         │ 开始录音    │            │            │            │          │          │
 │         ├───────────>│ 音频流      │            │            │          │          │
 │         │            ├───────────>│            │            │          │          │
 │         │            │            │ 累积音频    │            │          │          │
 │ 松开按钮 │            │            │            │            │          │          │
 ├────────>│            │            │            │            │          │          │
 │         │ end_speech │            │            │            │          │          │
 │         ├───────────>│            │            │            │          │          │
 │         │            │            │ 识别完成    │            │          │          │
 │         │            │<───────────┤            │            │          │          │
 │         │<───────────┤ 返回文本    │            │            │          │          │
 │         │            │            │            │            │          │          │
 │ 选择角色A│            │            │            │            │          │          │
 ├────────>│ request    │            │            │            │          │          │
 │         ├───────────>│            │            │            │          │          │
 │         │            │            │            │ 生成回答    │          │          │
 │         │            │            │            ├───────────>│          │          │
 │         │            │            │            │            │ API调用   │          │
 │         │            │            │            │            ├─────────────────────>│
 │         │            │            │            │            │          │  流式返回 │
 │         │            │            │            │            │<─────────────────────┤
 │         │<───────────┼────────────┼────────────┼────────────┤ 文本块    │          │
 │ 看到文本 │            │            │            │            │          │          │
 │         │            │            │            │            │          │          │
 │         │            │            │            │            │ TTS合成   │          │
 │         │            │            │            │            ├─────────>│          │
 │         │            │            │            │            │<─────────┤          │
 │         │<───────────┼────────────┼────────────┼────────────┤ 音频URL   │          │
 │ 听到语音 │ 播放音频    │            │            │            │          │          │
 │         │            │            │            │            │          │          │
 │         │            │            │            │ 预生成B、C  │          │          │
 │         │            │            │            ├───────────>│          │          │
 │         │            │            │            │            ├─────────────────────>│
 │         │            │            │            │            │<─────────────────────┤
 │         │            │            │            │            ├─────────>│          │
 │         │            │            │            │<───────────┤          │          │
 │         │            │            │            │ 缓存完成    │          │          │
```

**关键时间节点**：
- STT延迟：300-800ms（取决于语音长度）
- Claude API延迟：首token 500-1000ms，流式输出 50ms/token
- TTS延迟：2-3秒（100字）
- 总延迟（用户说完到听到AI回答）：3-5秒

**优化后延迟**：
- 预生成命中：<100ms（直接播放缓存音频）
- 流式文本显示：用户在等待音频时可先看到文字，感知延迟降低

---

## 4. 延迟优化策略

### 4.1 STT优化

| 策略 | 效果 | 实现方式 |
|------|------|----------|
| faster-whisper替代原版 | 速度提升4x | 使用CTranslate2量化推理 |
| VAD前置过滤 | 减少无效推理 | silero-vad过滤静音段 |
| 模型预热 | 消除首次冷启动 | 服务启动时推理一次空音频 |
| float16精度 | 速度提升2x | compute_type="float16" |

### 4.2 LLM优化

| 策略 | 效果 | 实现方式 |
|------|------|----------|
| Prompt Caching | 输入成本降低90% | system prompt加cache_control |
| 流式输出 | 感知延迟降低 | 边生成边显示文字 |
| 预生成 | 响应时间<100ms | 角色A发言后立即预生成B、C |
| 上下文压缩 | 减少token消耗 | 超过10轮后生成摘要 |

### 4.3 TTS优化

| 策略 | 效果 | 实现方式 |
|------|------|----------|
| 音频缓存 | 重复内容秒播 | MD5哈希缓存 |
| 分句流式 | 首句延迟降低 | 按标点分句，逐句合成播放 |
| 预生成 | 用户点击即播 | 后台并行合成所有角色音频 |
| 模型预热 | 消除冷启动 | 服务启动时合成一句话 |

### 4.4 端到端延迟目标

```
场景1：首次回答（无预生成缓存）
  STT:  0.5s
  LLM:  1.0s（首token）+ 流式显示文字
  TTS:  2.5s（100字）
  总计: ~4s（用户感知：1.5s后看到文字，4s后听到声音）

场景2：预生成命中
  检查缓存: <10ms
  播放音频: 立即
  总计: <100ms

场景3：用户追问（缓存失效）
  LLM:  1.0s（首token）
  TTS:  2.5s
  总计: ~3.5s
```

---

## 5. WebSocket API设计

### 5.1 消息协议

```typescript
// 前端 → 后端
type ClientMessage =
  | { type: "audio_chunk"; data: string }      // base64编码音频
  | { type: "end_speech" }                      // 用户停止说话
  | { type: "request_response"; character_id: string; followup?: string }
  | { type: "start_session"; session_config: SessionConfig }

// 后端 → 前端
type ServerMessage =
  | { type: "stt_result"; text: string }
  | { type: "text_chunk"; character_id: string; text: string }  // 流式文本
  | { type: "audio_ready"; character_id: string; audio_url: string }
  | { type: "pre_gen_ready"; character_ids: string[] }  // 预生成完成通知
  | { type: "error"; code: string; message: string }
  | { type: "state_change"; state: DialogueState }
```

### 5.2 FastAPI WebSocket端点

```python
# api/websocket.py

from fastapi import FastAPI, WebSocket, WebSocketDisconnect
import json

app = FastAPI()

@app.websocket("/ws/{session_id}")
async def websocket_endpoint(websocket: WebSocket, session_id: str):
    await websocket.accept()

    # 初始化会话组件
    stt = WhisperSTTService(STTConfig())
    context = ContextManager()
    characters = CharacterEngine()
    tts = CoquiTTSService()
    redis = await get_redis()
    manager = DialogueManager(context, characters, tts, redis)

    try:
        while True:
            raw = await websocket.receive_text()
            msg = json.loads(raw)

            if msg["type"] == "audio_chunk":
                audio = base64.b64decode(msg["data"])
                text = await stt.transcribe_stream(audio)
                if text:
                    await websocket.send_json({
                        "type": "stt_result", "text": text
                    })

            elif msg["type"] == "request_response":
                char_id = msg["character_id"]
                followup = msg.get("followup")

                # 流式发送文本
                async for chunk in manager.handle_user_input(
                    text=followup or context.last_user_text,
                    target_character_id=char_id,
                ):
                    await websocket.send_json(chunk)

            elif msg["type"] == "end_speech":
                stt.reset()

    except WebSocketDisconnect:
        pass
```

---

## 6. 部署方案

### 6.1 硬件要求

**最低配置（单机本地部署）**：

| 组件 | 最低要求 | 推荐配置 |
|------|----------|----------|
| GPU | NVIDIA RTX 3080 (10GB VRAM) | RTX 4090 (24GB VRAM) |
| CPU | 8核 | 16核 |
| 内存 | 32GB | 64GB |
| 存储 | 100GB SSD | 500GB NVMe SSD |
| 网络 | 100Mbps | 1Gbps |

**显存分配**：
```
Whisper large-v3:  ~6GB
Coqui XTTS-v2:    ~4GB
系统预留:          ~2GB
总计:              ~12GB（RTX 3080 10GB不够，需RTX 3090 24GB或4090）

降级方案（10GB显存）：
  Whisper medium:  ~2GB
  Coqui XTTS-v2:  ~4GB
  总计:            ~6GB（可用）
```

### 6.2 Docker容器化方案

```yaml
# docker-compose.yml

version: "3.9"

services:
  backend:
    build: ./backend
    runtime: nvidia
    environment:
      - NVIDIA_VISIBLE_DEVICES=all
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - REDIS_URL=redis://redis:6379
    volumes:
      - ./models:/app/models      # AI模型文件
      - ./audio_cache:/app/audio_cache
      - ./voices:/app/voices      # 音色样本
    ports:
      - "8000:8000"
    depends_on:
      - redis
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]

  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    environment:
      - VITE_WS_URL=ws://localhost:8000

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  redis_data:
```

```dockerfile
# backend/Dockerfile

FROM nvidia/cuda:12.1-cudnn8-runtime-ubuntu22.04

RUN apt-get update && apt-get install -y python3.11 python3-pip ffmpeg

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt

# 预下载模型（构建时缓存，避免运行时下载）
RUN python -c "from faster_whisper import WhisperModel; WhisperModel('large-v3', device='cpu')"

COPY . .
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 6.3 项目目录结构

```
project/
├── backend/
│   ├── api/
│   │   ├── websocket.py      # WebSocket端点
│   │   └── http.py           # REST API（角色配置管理）
│   ├── stt/
│   │   └── whisper_service.py
│   ├── character/
│   │   ├── engine.py
│   │   └── configs.py        # 角色System Prompt配置
│   ├── tts/
│   │   └── coqui_service.py
│   ├── dialogue/
│   │   ├── manager.py
│   │   └── context.py
│   ├── models/               # AI模型文件（gitignore）
│   ├── voices/               # 音色样本
│   ├── audio_cache/          # TTS缓存
│   ├── requirements.txt
│   └── main.py
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── AudioRecorder.tsx
│   │   │   ├── CharacterPanel.tsx
│   │   │   └── DialogueHistory.tsx
│   │   ├── hooks/
│   │   │   └── useWebSocket.ts
│   │   └── App.tsx
│   └── package.json
└── docker-compose.yml
```

---

## 7. 成本估算

### 7.1 硬件成本

| 方案 | 硬件 | 一次性成本 |
|------|------|-----------|
| 最低可用 | RTX 3090 + 32GB RAM + i7 | ~¥15,000 |
| 推荐配置 | RTX 4090 + 64GB RAM + i9 | ~¥30,000 |
| 云GPU（按需） | A10G实例（AWS g5.xlarge） | ~¥15/小时 |

### 7.2 API成本（Claude）

```
claude-sonnet-4-20250514 定价：
  输入：$3/M tokens
  输出：$15/M tokens
  缓存写入：$3.75/M tokens
  缓存读取：$0.30/M tokens

一场30分钟节目估算（50次API调用）：
  输入（含缓存）：~52,000 tokens ≈ $0.16
  输出：~10,000 tokens ≈ $0.15
  单场成本：~$0.31（约¥2.2）

月度成本（每天1场节目）：
  API费用：~¥66/月
  云服务器（如使用）：~¥500-2000/月
```

### 7.3 本地部署 vs 云部署对比

| 维度 | 本地部署 | 云部署 |
|------|----------|--------|
| 一次性成本 | ¥15,000-30,000 | ¥0 |
| 月运营成本 | ¥66（仅API） | ¥500-2000 |
| 延迟 | 最低（本地推理） | 较高（网络传输） |
| 维护成本 | 需自行维护 | 托管服务 |
| 扩展性 | 受硬件限制 | 弹性扩展 |
| 推荐场景 | 固定录制场地 | 多地点/多人使用 |

**推荐**：初期本地部署（控制成本），验证产品后再考虑云化。

---

## 8. 技术风险与缓解措施

| 风险 | 概率 | 影响 | 缓解措施 |
|------|------|------|----------|
| GPU显存不足 | 中 | 高 | 降级到medium模型；STT/TTS分时使用GPU |
| Claude API延迟波动 | 中 | 中 | 预生成策略；设置超时重试 |
| 角色"串戏"（角色混淆） | 中 | 中 | 强化System Prompt；上下文中明确标注发言者 |
| TTS中文发音错误 | 低 | 中 | 添加文本预处理（数字、英文转中文） |
| WebSocket断连 | 低 | 高 | 心跳检测；自动重连；会话状态持久化到Redis |
| 上下文超长 | 低 | 中 | 滑动窗口 + 摘要机制 |

---

## 9. 开发路线图

**Phase 1（2周）：核心链路验证**
- [ ] STT模块：Whisper集成，验证中文识别准确率
- [ ] LLM模块：Claude API集成，单角色对话
- [ ] TTS模块：Coqui集成，验证音色效果
- [ ] 基础WebSocket通信

**Phase 2（2周）：多角色与预生成**
- [ ] 多角色System Prompt设计与调优
- [ ] 对话管理器：状态机、上下文管理
- [ ] 预生成策略实现
- [ ] Redis缓存集成

**Phase 3（1周）：前端与体验优化**
- [ ] React前端：录音、角色选择、对话历史
- [ ] 流式文本显示
- [ ] 音频播放控制

**Phase 4（1周）：部署与测试**
- [ ] Docker容器化
- [ ] 端到端延迟测试与优化
- [ ] 压力测试（长时间录制稳定性）

---

## 10. 关键代码示例

### 10.1 主应用入口

```python
# backend/main.py

from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles
from contextlib import asynccontextmanager
import redis.asyncio as redis

# 全局服务实例（单例模式，避免重复加载模型）
stt_service = None
tts_service = None
character_engine = None
redis_client = None

@asynccontextmanager
async def lifespan(app: FastAPI):
    """应用生命周期管理：启动时加载模型，关闭时清理资源"""
    global stt_service, tts_service, character_engine, redis_client

    print("Loading AI models...")
    stt_service = WhisperSTTService(STTConfig())
    tts_service = CoquiTTSService()
    character_engine = CharacterEngine()
    redis_client = await redis.from_url("redis://localhost:6379")

    # 模型预热
    print("Warming up models...")
    await stt_service.transcribe_stream(b"\x00" * 16000)  # 1秒静音
    await tts_service.synthesize("测试", "male_energetic")

    print("Ready!")
    yield

    # 清理
    await redis_client.close()

app = FastAPI(lifespan=lifespan)

# 静态文件服务（音频缓存）
app.mount("/audio", StaticFiles(directory="audio_cache"), name="audio")

# WebSocket端点
from api.websocket import websocket_endpoint
app.add_api_route("/ws/{session_id}", websocket_endpoint)

# REST API（角色配置管理）
from api.http import router as http_router
app.include_router(http_router, prefix="/api")
```

### 10.2 前端WebSocket Hook

```typescript
// frontend/src/hooks/useWebSocket.ts

import { useEffect, useRef, useState } from 'react';

interface Message {
  type: string;
  [key: string]: any;
}

export function useWebSocket(url: string) {
  const ws = useRef<WebSocket | null>(null);
  const [messages, setMessages] = useState<Message[]>([]);
  const [isConnected, setIsConnected] = useState(false);

  useEffect(() => {
    ws.current = new WebSocket(url);

    ws.current.onopen = () => {
      setIsConnected(true);
      console.log('WebSocket connected');
    };

    ws.current.onmessage = (event) => {
      const msg = JSON.parse(event.data);
      setMessages((prev) => [...prev, msg]);
    };

    ws.current.onclose = () => {
      setIsConnected(false);
      console.log('WebSocket disconnected');
      // 自动重连
      setTimeout(() => {
        window.location.reload();
      }, 3000);
    };

    return () => {
      ws.current?.close();
    };
  }, [url]);

  const sendMessage = (msg: Message) => {
    if (ws.current?.readyState === WebSocket.OPEN) {
      ws.current.send(JSON.stringify(msg));
    }
  };

  const sendAudioChunk = (audioData: ArrayBuffer) => {
    const base64 = btoa(
      String.fromCharCode(...new Uint8Array(audioData))
    );
    sendMessage({ type: 'audio_chunk', data: base64 });
  };

  return { messages, isConnected, sendMessage, sendAudioChunk };
}
```

### 10.3 前端录音组件

```typescript
// frontend/src/components/AudioRecorder.tsx

import { useState, useRef } from 'react';

interface Props {
  onAudioChunk: (data: ArrayBuffer) => void;
  onRecordingEnd: () => void;
}

export function AudioRecorder({ onAudioChunk, onRecordingEnd }: Props) {
  const [isRecording, setIsRecording] = useState(false);
  const mediaRecorder = useRef<MediaRecorder | null>(null);
  const audioContext = useRef<AudioContext | null>(null);

  const startRecording = async () => {
    const stream = await navigator.mediaDevices.getUserMedia({
      audio: {
        sampleRate: 16000,
        channelCount: 1,
        echoCancellation: true,
        noiseSuppression: true,
      },
    });

    audioContext.current = new AudioContext({ sampleRate: 16000 });
    const source = audioContext.current.createMediaStreamSource(stream);

    // 使用ScriptProcessor实时发送音频块
    const processor = audioContext.current.createScriptProcessor(4096, 1, 1);
    processor.onaudioprocess = (e) => {
      const inputData = e.inputBuffer.getChannelData(0);
      // 转换为16-bit PCM
      const pcm = new Int16Array(inputData.length);
      for (let i = 0; i < inputData.length; i++) {
        pcm[i] = Math.max(-32768, Math.min(32767, inputData[i] * 32768));
      }
      onAudioChunk(pcm.buffer);
    };

    source.connect(processor);
    processor.connect(audioContext.current.destination);

    setIsRecording(true);
  };

  const stopRecording = () => {
    audioContext.current?.close();
    setIsRecording(false);
    onRecordingEnd();
  };

  return (
    <button
      onMouseDown={startRecording}
      onMouseUp={stopRecording}
      onTouchStart={startRecording}
      onTouchEnd={stopRecording}
      className={isRecording ? 'recording' : ''}
    >
      {isRecording ? '松开结束' : '按住说话'}
    </button>
  );
}
```

---

## 11. 扩展性考虑

### 11.1 水平扩展方案

当单机性能不足时，可采用以下扩展策略：

```
                    ┌─────────────┐
                    │   Nginx     │
                    │ Load Balancer│
                    └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
         ┌────────┐   ┌────────┐   ┌────────┐
         │Backend1│   │Backend2│   │Backend3│
         │(GPU 1) │   │(GPU 2) │   │(GPU 3) │
         └────┬───┘   └────┬───┘   └────┬───┘
              │            │            │
              └────────────┼────────────┘
                           ▼
                    ┌─────────────┐
                    │    Redis    │
                    │  (共享缓存)  │
                    └─────────────┘
```

**关键点**：
- WebSocket会话粘性：使用Nginx的`ip_hash`或`sticky session`
- 共享缓存：预生成结果存入Redis，任意节点可读取
- 模型复制：每个节点独立加载AI模型

### 11.2 功能扩展方向

1. **多语言支持**：Whisper和XTTS都支持多语言，只需调整配置
2. **实时字幕**：STT结果实时显示，支持听障用户
3. **对话导出**：生成播客音频文件和文字稿
4. **角色自定义**：用户上传音色样本，自定义角色人设
5. **观众互动**：观众通过弹幕提问，主持人选择性回答

---

## 12. 总结

### 12.1 架构亮点

1. **成本优化**：本地STT/TTS + Claude API + Prompt Caching，单场节目成本<¥3
2. **低延迟**：预生成策略 + 流式处理，响应时间<100ms（缓存命中）
3. **可扩展**：模块化设计，支持水平扩展和功能迭代
4. **易部署**：Docker一键部署，降低运维门槛

### 12.2 技术选型理由总结

| 组件 | 选型 | 核心理由 |
|------|------|----------|
| STT | faster-whisper | 免费、准确、速度快4倍 |
| LLM | Claude Sonnet 4 | 性价比最优、Prompt Caching |
| TTS | Coqui XTTS-v2 | 免费、音色克隆、中文支持 |
| 后端 | Python + FastAPI | AI生态最佳、异步高性能 |
| 通信 | WebSocket | 双向低延迟、音频流支持 |
| 缓存 | Redis | 高性能、发布订阅、分布式 |

### 12.3 下一步行动

1. **技术验证**（1周）：搭建最小原型，验证STT→LLM→TTS链路
2. **角色调优**（1周）：设计并测试3个角色的System Prompt，确保风格差异明显
3. **性能测试**（3天）：压测端到端延迟，优化至目标值
4. **用户测试**（1周）：邀请目标用户试用，收集反馈

---

## 附录：依赖清单

```txt
# backend/requirements.txt

fastapi==0.109.0
uvicorn[standard]==0.27.0
websockets==12.0
python-multipart==0.0.6

# AI模型
faster-whisper==1.0.0
anthropic==0.18.0
TTS==0.22.0  # Coqui TTS

# 数据处理
numpy==1.26.3
torch==2.2.0
torchaudio==2.2.0

# 基础设施
redis[hiredis]==5.0.1
aioredis==2.0.1
sqlalchemy==2.0.25
alembic==1.13.1

# 工具
python-dotenv==1.0.0
pydantic==2.5.3
pydantic-settings==2.1.0
```

```json
// frontend/package.json
{
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "typescript": "^5.3.3",
    "@types/react": "^18.2.48",
    "vite": "^5.0.11"
  }
}
```

---

**文档版本**：v1.0  
**最后更新**：2026-04-20  
**作者**：AI架构师  
**审核状态**：待技术评审
