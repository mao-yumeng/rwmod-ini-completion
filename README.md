Rusted Warfare mod ini
给 Rusted Warfare 模组的 .ini 文件提供自动补全和悬停提示的 VS Code 扩展。
RW 的 ini 是全有或全无的：一个未知音效名、一个放错段的键、一张不存在的贴图， 都会让整个模组加载失败，而报错往往只说"哪个文件的哪一行"，不说"应该是什么"。 这个扩展的目标就是让你在写的时候就不会写错。
功能
位置	补什么
行首输入 [	227 种官方段名（core、turret_#、action_*、hiddenAction_* …）
段内输入键名	只补该段合法的键，按官方出现频率排序
键: 之后	枚举值、音效名、贴图名、单位 id、effect 名
悬停键名	官方出现次数、可用段、可选值
悬停段名	该段的全部合法键
为什么只补"该段的键"
因为段位放错是硬失败。例如 showEnergyBar 属于 [graphics]，写进 [core] 会让 游戏报 The key '[core]showEnergyBar' was not used 并拒绝加载整个模组。
所以当段名已知时，扩展只给出该段出现过的键，不做兜底——宁可少补几个， 也不给一个会导致加载失败的选项。
只有当段名完全未知（你自己新造的段）时，才退回补全全部键。
会主动兜住的几类错误
音效键（shoot_sound、soundOnDeath …）只补 30 个真实内置音效 + 模组自己的 ROOT:sound/*.wav。写错名字会报 Could not find sound:xxx。
贴图键（image、image_turret …）只补 SHARED: 下真实存在的文件 + 当前目录的 png。
枚举键（270 个）直接给官方用过的取值，不用查文档。
安装
方式一：VSIX
code --install-extension rwmod-ini-0.1.0.vsix
或 VS Code 里 扩展 → ... → 从 VSIX 安装。
方式二：开发模式
git clone <this-repo>
cd rwmod-ini
code .
# 按 F5，会打开一个加载了本扩展的新窗口
方式三：直接放进扩展目录
把整个目录复制到 ~/.vscode/extensions/rwmod-ini-0.1.0/，重启 VS Code。
使用前提
扩展只在「看起来像 RW mod」的 ini 里生效，判定条件是文件里出现以下任一：
一个 [core] 段头
class: CustomUnitMetadata
displayText: 键
这样不会污染普通 ini 文件（git config、各类编辑器配置等）。
数据从哪来
data/rwmod-lang.json 是从游戏本体扫描生成的，不是手写的：
内容	来源
段名、键、键所属段、取值枚举	官方 assets/**/*.ini（132 个文件）
内置音效名、内置效应名、音效类键	game-lib.jar 常量池
共享贴图文件名	assets/units/shared/*.png
单位 id	官方单位 + 可选的模组目录
当前数据规模：227 段 / 538 键 / 270 个枚举键 / 30 音效 / 215 单位 / 48 共享贴图。
重新生成
游戏更新后，用你自己的游戏目录重新生成：
node scripts/gen-lang-data.js --assets "E:/Rusted/Rusted Warfare/assets"
如果还想把某个模组自己的单位 id、音效、effect 也补进去：
node scripts/gen-lang-data.js \
  --assets "E:/Rusted/Rusted Warfare/assets" \
  --mod "F:/my-mod/喵石单位"
不传 --assets 时会尝试几个常见 Steam 安装路径。
重新生成后，在编辑器里执行命令 RW mod: 重新加载补全数据（或重启 VS Code）。
开发
node test/smoke.js     # 无头冒烟测试，14 项断言
测试通过桩掉 vscode 模块，直接调用补全函数，验证： 段名补全、段内键过滤（[graphics] 里不能出现 shoot_sound）、 音效/枚举/贴图/单位 id 补全、RW ini 判定、段名归一化。
extension.js 末尾导出的 __test 就是给这个测试用的，VS Code 运行时不走那条路径。
已知限制
只做补全与悬停，不做错误检查。真正的加载前校验建议另外跑脚本扫一遍 （段位、引用存在性、音效名、BOM）。
表达式语言不做补全。RW 的条件表达式（if self.hp(lessThan=1)、@define、 ${} 插值、随机函数）词表只能从 game-lib.jar 反推，官方无文档、语法未完全验证， 补上去只会给错的建议，所以刻意不做。
段名已知时严格不给跨段键。如果你确定某个跨段用法合法（RW 偶尔允许）， 悬停可以看到该键的「可用段」列表。
License
MIT
