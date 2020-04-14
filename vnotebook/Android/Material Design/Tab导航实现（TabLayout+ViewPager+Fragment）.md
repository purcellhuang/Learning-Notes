# Tab导航实现（TabLayout+ViewPager+Fragment）
## 1. 概念介绍
### 1.1 TabLayout
* 定义：实现Material Design效果的控件库（Android Design Support Library）；
* 作用：用于实现点击选项进行切换选项卡的自定义效果（5.0可用）
### 1.2 ViewPager
* 定义：ViewPager是android扩展包v4包中的类
* 作用：左右切换当前的view，实现滑动切换的效果。
### 1.3 Fragment
* 定义：Fragment是activity的界面中的一部分或一种行为
* 作用：支持更动态、更灵活的界面设计（从3.0开始引入）
## 2. 实现步骤
* 步骤1：添加依赖
* 步骤2：创建需要的Fragment布局文件（需要多少个Tab选项，就建多少个Fragment）
* 步骤3：创建Fragment对应的Activity类
* 步骤4：定义适配器Adapter
* 步骤5：定义主布局activity_main.xml文件
* 步骤6：定义MainActivity类

