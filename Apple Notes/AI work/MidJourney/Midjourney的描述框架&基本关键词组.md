**01.描述框架** 1.框架讲解 2.基础属性详解 **02.基本词组**
 **01.描述框架** 本“描述框架”不唯一，可根据个人喜好进行调整取舍，仅作为推荐

## ***参考图+场景框架+画风模型选择+主体+详情描述+基础属性***

**参考图：** 参考图（贴图），目前最多可以喂给AI 30张参考图。AI会分析你的参考图获取相应元素应用于即将产出的画面中。 参考图格式：png/jpg

**场景框架：** 画面视角（正视图、侧视图、俯视图、仰视图、顶视图等）、灯光角度及冷暖（角度：逆光、顶光、侧光、轮廓等，冷暖：太阳光、冷光、暖光、赛博朋克光等）、构图（居中构图、对称构图、三分法构图、S型构图、对角线构图、水平构图）、拍摄仪器等。

**画风模型选择：** 风格（古典主义、新古典主义、浪漫主义、现实主义、印象主义、后印象主义、立体主义、抽象主义、达达主义、现代主义和未来主义等）；类型（水墨画、油画、版画、水粉画、壁画、漫画、工笔画、写意画、抽象画、绿色山水画、水墨山画、素描等）。

**主体：** 告诉AI画的内容，比如猫咪、摩托车、建筑等。

**详情描述：** 在做什么事、什么样的动作、穿着什么样式的衣着、什么颜色、什么情绪、什么发饰等细节。

**基础属性：** 画面尺寸比例、模型选择、画面质量、风格变化程度、占比权重、排除项、画面微调、无缝贴图单元、风格值、参考图参考值等。

## **基础属性详解**

**1.画面尺寸比例：** --aspect 1:1 默认纵横比。 --aspect 5:4 常见的框架和打印比例。 --aspect 3:2 印刷摄影中常见。 --aspect 7:4 靠近高清电视屏幕和智能手机屏幕。 --ar X:X 画面比例，支持1:2/3:2/16:9 (v3模型下--ar16:9（1.78）实际被渲染为7:4（1.75）；v4模型目前只支持1:1和3:2（2:3）若不输入--ar则默认输出图像为1:1的正方形)  **2.模型选择：** v4是最新版本，效果最好（v5即将发行）。「直接打--v4，系统会自动加上style 4b；直接打--v4 --style 4a另外一种选择」niji动漫风格。  **3.画面质量：** --q .25/.5/1/2四种 数值越大不代表像素越大，数字越大，画面质量大，渲染时间多久。根据自身需求主题选择，突出结构线条可选择q大一些。「--v4默认提升器用的较多，增加画像大小，同时平滑或细化细节；--upanime是niji模型默认的提升器，当你加上之后，点击U按钮会使用这个提升器。-- upbeat是提升画像光滑度。-- uplight是添加少量纹理」

**4.风格变化程度：** --chaos <0~100>,可简写--c<0~100> v4模型中，数值越大，风格变化越多；数值越小，风格越统一。主要控制出图的多样性。默认值--chaos0

**5.占比权重：** ccc::xxx 英文的双冒号为权重分割符号。例：bule shirt蓝色衬衫 ，被分割后bule::shirt 可理解为一件蓝色的衬衫，加入数量修饰词。ccc::xxx默认为前后属性词在画面中占比为1(ccc):1(xxx)。ccc::1.5xxx可翻译成ccc在画面中的占比为xxx的1.5倍。(此处占比关系为相对关系，表示占比程度，非画面中真实的占比)注：v1~v3模型不支持小数点，但v4模型支持，另，每个部分权重加起来总和必须是正数（例：bule::shirt、bule::1shirt::1、bule::1.6shirt::0.4、bule::20shirt::40）

**6.排除项：** --no 修饰词 表述不要出现修饰词相应描述。例：--no car，则不会出现car。修饰词代表：负面/反向关键词，效果等同于权重分割符号加上数值-0.5.

**7.画面微调：** --seed <0~4294967295> 用于微调已经生成的画面，获取相应seed数值后，微调描述词组即可。系统默认值seed是一个随机数值。在v4和niji模型中，相同的seed数值下，相同的描述词组（空格、标点等），会产生完全一样的4张初始图片。注：seed数值需要获取初始图数值，如果获取应用提升器后的单张图片seed数值，此seed数值无意义。

**8.无缝贴图单元：** --tile 用于生产可无缝连接的贴图单元（生成的图片可无缝连接铺满屏幕）。--tile目前支持v1～v3模型和test/testp模型（v5模型上线后也支持哦～）。例：描述 --v3 --tile、描述 --test --tile、描述 --testp --tile。（无缝贴图检查器：[https://www.pycheung.com/checker/](https://link.zhihu.com/?target=https%3A//www.pycheung.com/checker/)）

**9.风格值：** --stylize+数值 数值越大生成画像与描述关联越小，数值越小生成画像与描述关联越大。v4模型中，默认数值为100，取值范围0～1000；v3模型中，取值范围625～60,000;niji模型无法调整。

**10.参考图参考值：** --iw .25/.5/1~5 数值越大参考图选取的元素样式越大，画像结果与参考图效果越接近。系统默认数值0.25，最大值5。

## **02.基本词组**

情绪词组 **高兴：**happy **高兴地：**elated **充满希望地：**hopeful **喜悦的、开心的：**amused **惊呀的：**dsurprised **满意的、满足的：**content **兴奋的、极其激动的：**excited **恼怒的：**irritated **生气、愤怒：**angry **沮丧：**depressed **难过：**upset **忧愁：**sad **焦虑：**anxious **担忧的：**worried **害怕：**afraid **厌恶：**disgusted **恐惧的：**fearful **憎恨的：**hateful

构图词组 **画面视角**（正视图、侧视图、俯视图、仰视图、顶视图等） **正视图：**front view **侧视图：**side view **俯视图：**bird view **仰视图：**upward view **顶视图：**top view **广角视图：**wide view **产品视图：**product view **超广角：**ultrawide shot **中景：**medium shot **特写：**headshot **微距拍摄：**macro shot **肖像：**portrait **全身像：**full boby **半身像：**busts **对称脸：**symmetrical face **对称身体：**symmetrical boby  **灯光角度及冷暖**（角度：逆光、顶光、侧光、轮廓光等，冷暖：太阳光、冷光、暖光、赛博朋克光等） **逆光：**back light **顶光：**top light **侧光：**raking light **轮廓光：**rim light **边缘光：**edge light **强光：**hard light **反光：**reflection light **自然光：**natural light **立体光：**volumetric light **电影光：**studio light **戏剧光：**dramatic light **太阳光：**sun light **漂亮光：**beautiful light **柔和光：**soft light **影视光：**studio light **晨光：**morning light **黄昏：**crepuscular Ray **黄金时光：**golden hour light **映射光：**mapping light **情调光：**mood light **氛围光：**atmospheric light **冷光：**cold light **暖光：**warm light **赛博朋克光：**cyberpunk light **伦勃朗光：**rembrandt light **彩色光：**color light  **构图**（居中构图、对称构图、三分法构图、S型构图、对角线构图、水平构图） **居中构图：**center the composition **对称构图：**symmetrical the composition **三分法构图：**rule of thirds composition **s型构图：**s-shaped composition **对角线构图：**diagonal composition **水平构图：**horizontal composition **曼陀罗构图：**mandala

风格词组 **风格（**古典主义、新古典主义、浪漫主义、现实主义、印象主义、后印象主义、立体主义、抽象主义、达达主义、现代主义和未来主义等） **古典主义：**classicism **新古典主义：**neoclassicism **浪漫主义：**romanticism **现实主义：**realism **超现实主义：**surrealism **新现实主义：**neo-realism **印象主义：**impressionism **后印象主义：**post-impressionism **立体主义：**cubism **抽象主义：**abstractionism **达达主义：**dadaism **现代主义：**modernism **未来主义：**futurism **浮世绘：**ukiyoe **概念艺术：**trending on dixiy concept art **潜意识：**subconsciousness **创世纪风：**genesis **废土朋克：**wasteland punk **赛博朋克：**cyber punk **海报风格：**poster style **数字雕刻风格：**digitally engraved **建筑设计风格：**architectural design **名人风格：**希施金风格（Ivan I. Shishkin）、宫崎骏风格（studio ghibli）等 **A站风格：**trending on artsation  **类型**（水墨画、油画、版画、水粉画、壁画、漫画、工笔画、写意画、抽象画、绿色山水画、水墨山画、素描等） **水墨画：**ink style **油画：**oil painting **版画：**engraving **水粉画：**gouache **壁画：**mural **漫画：**cartoon **工笔画：**gongbi painting、Chinese elaborate-style painting **写意画：**freehand painting **抽象画：**abstract painting **写实画：**realistic painting **绿色山水画：**green landscape painting **水墨山画：**ink mountain paintin **景观画：**landscape **素描：**sketch **黑白画：**black and white **水彩画：**watercolor painting **原画：**original **插画：**illustration **利诺剪裁：**lino cut

色彩词组 **白色：**white **黑色：**black **绿色：**green **红色：**red **蓝色：**blue **紫色：**purple **黄色：**yellow **橙色：**orange **褐色：**tan **青色：**syan **灰色：**gray **棕色：**brown

色调词组 **单色调：**monotone **马卡龙色调：**Macaron tone **多彩色调：**colourful color matching **淡色色调：**muted tone **霓虹灯色调：**neon shades **高纯度色调：**the high-purity tone **低纯度色调：**the low-purity tone **黑背景色调：**black background centered **红黑色调：**red and black tone **黄黑色调：**yellow and black tone **白色粉色色调：**white and pink tone **金银色调：**gold and silver tone

质感词组 **羊绒：**cashmere **彩绘玻璃：**stained glass **羽毛：**feather **镭射：**holographic **毛绒感：**fluffy

元素词组（可依据：饰品、配饰、点缀、国家等多维度展开） **相机：**camera **麦克风：**microphone **武器：**weapons **刀：**sword **镰刀：**scythe **枪：**gun **魔杖：**wand **伞：**unbrella **龙：**loong **凤凰：**chiness phoenix **麒麟：**kylin **玉：**jade

渲染词组 **虚幻引擎：**unreal engine **oc渲染：** oc rendlerer **建筑渲染：** architectural visualisation **室内渲染：** Corona Render **真实感：** Quixel Megascans Render **v射线：** V-Ray