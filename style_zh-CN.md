```eval_rst
.. include:: /header.rst 
:github_url: |github_link_base|/overview/style.md
```
# 样式

*样式* 被用于设置对象的外观。lvgl 的样式系统受启发于 CSS. 简而言之，其概念如下所示：

- 样式是一个 `lv_style_t` 属性，它能保存边框宽度，文本颜色等属性，它类似于 CSS 里面的 `class` 

- 对象能够改变样式来改变外观，在赋值过程中，可以指定目标部分(*pseudo-element* in CSS)和目标状态(*pseudo class*) 

- 
  例如在 Slider 控件处于按下状态时，可以将 `lv_style_blue` 属性添加到 Slider 控件的滑块上

- 样式可以级联，这意味着一个 object 可以使用多个样式，并且每个一个样式都有不同的属性。因此，并非所有的属性都需要在样式中指定，LVGL 将会寻找属性表去定义样式，如果没找到则使用 default 值。例如 `style_btn` 会让 button 的颜色使用默认的灰色，`style_btn_red` 只能添加一个 `background-color=red` 去覆盖背景颜色

- 后添加的样式具有更高的优先级，这意味着如果在两个样式指定了一个属性，则只有后一个添加的样式才会被使用

- 很多属性（例如 text color ）在对象中没有被指定时能够继承自其父控件

- 对象能够拥有比 "normal" 样式更高的优先级的本地样式

- 不同于 CSS，在 LVGL 中的一个属性控制一个状态

- 当 object 改变状态时就可以应用 Transitions

  


## 状态
对象可以处于以下状态的组合：

- `LV_STATE_DEFAULT` (0x0000) Normal, released 状态
- `LV_STATE_CHECKED` (0x0001) Toggled or checked 状态
- `LV_STATE_FOCUSED` (0x0002)通过键盘或者编码器聚焦或者触控板/鼠标点击
- `LV_STATE_FOCUS_KEY` (0x0004) 通过键盘或者编码器但不通过触控板/鼠标来聚焦 
- `LV_STATE_EDITED` (0x0008) 由编码器编辑
- `LV_STATE_HOVERED` (0x0010) 鼠标指针进入时的状态 (不支持现在)
- `LV_STATE_PRESSED` (0x0020) 开始触摸
- `LV_STATE_SCROLLED` (0x0040) 开始滑动
- `LV_STATE_DISABLED` (0x0080) Disabled 状态
- `LV_STATE_USER_1` (0x1000)  用户状态
- `LV_STATE_USER_2` (0x2000) 用户状态
- `LV_STATE_USER_3` (0x4000) 用户状态
- `LV_STATE_USER_4` (0x8000) 用户状态

该组合状态表示对象可以同时按下和聚焦。表示为 `LV_STATE_FOCUSED | LV_STATE_PRESSED`

样式可以添加任意的状态和状态组合。

例如，在 default 和 press 状态下分别设置不同的背景颜色。

如果属性未在状态中定义，那么将会使用最契合这个状态的属性。通常这意味着使用带有 `LV_STATE_DEFAULT` 的属性。

如果这个即使在默认状态下也没有设置属性，则将使用默认值（见后面）

但是什么是“最契合的状态属性”呢

状态的优先级依据上面的值，值越大，优先级越高。

为了确定使用哪个状态的属性，让我们举一个例子。想象一下，背景颜色被定义成以下样子：

- `LV_STATE_DEFAULT`: 白色
- `LV_STATE_PRESSED`: 灰色
- `LV_STATE_FOCUSED`: 红色

1. 在默认情况下，对象处于默认状态，所以这是一个简单情况：属性在对象的当前状态中被完美的定义为白色

2. 当对象处于 pressed 状态时，有两个相关属性：default 是白色（default 与每个状态相关），pressed 为灰色。pressed 优先级为 0x0020高于 default 的优先级 0x0000。故在按下时会使用灰色

3. 当对象处在 focused 状态时，会发生与 pressed 状态相同的一些事件，但是会用使用红色。（Focused 状态的优先级高于 default 状态的优先级）

4. 当对象处在 focused 和 pressed 时，灰色和红色同时会起作用，但是 pressed 状态的优先级高于 focused 状态的优先级，所以会使用灰色。

5. 可以设置成例如 `LV_STATE_PRESSED | LV_STATE_FOCUSED` 的玫瑰色

   在这种状态下，这个联合状态的的优先级为 0x0020 + 0x0002 = 0x0022，它高于 pressed 的优先级，所以会使用玫瑰色

6. 当对象在 checked 状态时，这个状态没有属性去设置背景颜色。因为没有更好的选择，所以对象从 default 状态的属性保持白色

一些使用的笔记:
- 状态的优先级非常直观, 这是用户天热期望的。例如，如果一个对象的状态是 focused 的，一个用户依旧想要看它是否是被按下，这里 pressed 状态有更高的优先级。

- 如果 focused 状态有更高的优先级，则它会覆盖 pressed 状态的颜色

- 如果你想要将所有状态设置会一个属性，只需要将其设 default 状态即可。如果这个对象不能找到这个属性的当前状态它将会去回退使用 default 状态的属性

- 使用 或运算符 可以将多个属性混合在一起使用（例如：pressed + checked + focused）

- 为不同的状态对应不同的样式元素是一个好主意

  例如：
  为 released, pressed, checked + pressed, focused, focused + pressed, focused + pressed + checked, 等状态寻找背景色是想当困难的

  相反，例如使用背景色来区分 pressed 和 checked 状态，用不同的边框颜色表示 focused 状态

## 级联样式
不需要在一种样式中设置所有的属性。可以让对象添加更多的样式，让后添加的样式修改或者拓展外观

例如：创建一个通用的灰色 button 样式并创建一个新的红色 button，其中只设置了新的背景颜色。这非常像 CSS 中当使用类列出 `<div class=".btn .btn-red">` 时。后添加的样式优先级高于前面添加的样式。所以在这个 灰色/红色 的示例中，首先添加正常 button 样式后添加红色的样式。然而来自状态的优先级依旧会生效。所以会产生以下两个情况

- 基础 button 样式定义的 default 状态使用 dark-gray 颜色，pressed 状态使用 light-gray 颜色
- 红色 button 样式定义在 default 状态时的背景颜色会红色

在这个条例中，当 button 处于 released 时（为 default 状态），它将是红色的，因为在最近添加的样式（red）找到了最佳匹配。当 button 在 pressed 时（为 pressed 状态），light-gray 是最好的匹配，因为它完美的描述了当前状态，所以 button 将会显示 light-gray

## 继承
很多属性（通常是与文本相关的那些）能够继承自其父对象的样式。

继承一般是在其对象的样式中没有去设置这些属性时生效（即使是 default 状态）

在这条，如果属性是继承的，则在父对象中检索到该值，知道它被另一个对象设置。父对象会根据自己本身的状态去设定这些值

所以如果一个 button 是 pressed 状态，且文本颜色来自继承，将会使用按下的文本颜色


## parts
对象可以有 parts，且 parts 可以有自己的样式 

下面列出在 LVGL 中的 parts：

- `LV_PART_MAIN` 类似矩形的背景颜色 
- `LV_PART_SCROLLBAR`  滑条
- `LV_PART_INDICATOR` 指示器, e.g. for slider, bar, switch, or the tick box of the checkbox
- `LV_PART_KNOB` Like a handle to grab to adjust the value*/
- `LV_PART_SELECTED` 指示当前选择的选项或部分
- `LV_PART_ITEMS`  Used if the widget has multiple similar elements (e.g. table cells)*/
- `LV_PART_TICKS` Ticks on scales e.g. for a chart or meter
- `LV_PART_CURSOR` 标记一个特定的地方。 例如：文本区域或者图标里的光标
- `LV_PART_CUSTOM_FIRST` 用户自定义的 parts 可以从这里添加。


For example a [Slider](/widgets/core/slider) has three parts:
- 背景色
- 指示器
- Knob

这意味着 Slider 总共有三个 parts 可以有它自己的样式，接下来看如何为对象或者 parts 添加样式



## 初始化样式和设置/获取属性 

样式储存在 `lv_style_t` 变量中。样式变量应该是 `static` 的。局的或动态分配的。换句话说，它们不能是函数内的局部变量，因为在函数退出时他们会被销毁。

在使用样式之前应该先用`lv_style_init(&my_style)` 初始化该样式，在初始化样式之后就可添加或者设置属性了。
属性的设置函数看起来像这样：`lv_style_set_<property_name>(&style, <value>);`例如：

```c
static lv_style_t style_btn;
lv_style_init(&style_btn);
lv_style_set_bg_color(&style_btn, lv_color_grey());
lv_style_set_bg_opa(&style_btn, LV_OPA_50);
lv_style_set_border_width(&style_btn, 2);
lv_style_set_border_color(&style_btn, lv_color_black());

static lv_style_t style_btn_red;
lv_style_init(&style_btn_red);
lv_style_set_bg_color(&style_btn_red, lv_color_red());
lv_style_set_bg_opa(&style_btn_red, LV_OPA_COVER);
```

删除属性使用：

```c
lv_style_remove_prop(&style, LV_STYLE_BG_COLOR);
```

从样式中获取一个属性的值：

```c
lv_style_value_t v;
lv_res_t res = lv_style_rget_prop(&style, LV_STYLE_BG_COLOR, &v);
if(res == LV_RES_OK) {	/*Found*/
	do_something(v.color);
}
```

`lv_style_value_t` 有三个区域:

- `num` 有整形, 布尔类型和不透明度属性
- `color` 颜色属性
- `ptr` 指针属性

重置样式（释放所有的样式数据），请使用

```c
lv_style_reset(&style);
```



## 在 widget 里面添加和删除样式

单独的一个样式没什么作用，需要分配给一个对象才能生效

### 添加样式

用 `lv_obj_add_style(obj, &style, <selector>)` 为一个对象添加样式。`<selector>` 是应添加 part 和 状态的 OR ed 值。例如：

- `LV_PART_MAIN | LV_STATE_DEFAULT`
- `LV_STATE_PRESSED`: The main part in pressed state. `LV_PART_MAIN` can be omitted
- `LV_PART_SCROLLBAR`: The scrollbar part in the default state. `LV_STATE_DEFAULT` can be omitted.
- `LV_PART_SCROLLBAR | LV_STATE_SCROLLED`: The scrollbar part when the object is being scrolled
- `0` Same as `LV_PART_MAIN | LV_STATE_DEFAULT`. 
- `LV_PART_INDICATOR | LV_STATE_PRESSED | LV_STATE_CHECKED` The indicator part when the object is pressed and checked at the same time.

Using `lv_obj_add_style`: 
```c
lv_obj_add_style(btn, &style_btn, 0);      				  /*Default button style*/
lv_obj_add_style(btn, &btn_red, LV_STATE_PRESSED);  /*Overwrite only a some colors to red when pressed*/
```

### 删除样式

使用 `lv_obj_remove_style_all(obj)` 来删除一个对象的所有样式。

使用 `lv_obj_remove_style(obj, style, selector)` 删除部分样式。这个函数仅会删除使用 `lv_obj_add_style` 添加进 `selector` 的样式

`style` 的值可以为 `NULL`，仅仅去匹配 `selector` 部分的值来删除样式，`selector` 可以使用 `LV_STATE_ANY` 和 `LV_PART_ANY` 这两个值来删除样式的所有状态和 part。 

### 样式改变回报

如果已经分配给对象的样式发生了改变（即添加或者改变一个属性），则使用了这个样式的对象将会收到通知。这里有 3 个选项可以执行此操作：

 	1. 如果您知道更改的属性可以通过简单的重绘（例如颜色或者透明度的改变）来实现，只需调用`lv_obj_invalidate(obj)` 或者`lv_obj_invalideate(lv_scr_act())`。
 	2. 如果改变或添加了更复杂的样式属性，并且您知道哪些对象收到这些样式的影响您可以调用`lv_obj_refresh_style(obj, part, property)`。刷新所有的 parts 和 属性使用`lv_obj_refresh_style(obj, LV_PART_ANY, LV_STYLE_PROP_ANY)`。
 	3. 要让 LVGL 检查所有对象以查看它们是否使用样式并在需要的时候刷新它们，请调用`lv_obj_report_style_change(&style)`。如果 `style` 部分为 `NULL` 时，所有的对象将会收到这个样式改变的通知。

### 获取对象的属性值
获取一个属性的最终值 - 考虑级联，继承，本地样式和 trainstations，可以使用这样的函数来获取：`lv_obj_get_style_<property_name>(obj, <part>)` 这个函数使用对象的当前状态，如果没有更好的候选者将返回默认值。例如：

```c
lv_color_t color = lv_obj_get_style_bg_color(btn, LV_PART_MAIN);
```

## 本地属性
此外，"normal" 样式，对象也可以储存本地样式。这个概念类似于 CSS 中的内联样式（例如，`<div style="color:red">`），但做了一些修改。

所以本地样式是类似于默认样式，但是它们不能在其它对象之间分享。如果这样使用了，本地对象会自动申请空间并在对象删除时释放。

它们有利于向对象添加本地定制。
不同于 CSS，在 LVGL 里面的本地样式可以分配给状态和 parts。

设置本地属性使用函数： `lv_obj_set_style_local_<property_name>(obj, <value>, <selector>);`

例如：

```c
lv_obj_set_style_local_bg_color(slider, lv_color_red(), LV_PART_INDICATOR | LV_STATE_FOCUSED);
```
## 属性
For the full list of style properties click [here](/overview/style-props).

### 典型的背景属性

在 widget 文档李你可以看见这样的的描述：“widget 使用典型的背景属性”。这个 "典型的背景属性" 与以下相关：

- Background
- Border
- Outline
- Shadow
- Padding
- Width and height transformation
- X and Y translation


## Transitions
By default, when an object changes state (e.g. it's pressed) the new properties from the new state are set immediately. However, with transitions it's possible to play an animation on state change.

假设，在一个对象改变状态时（例如，对象被按下了）立即设置新属性的新状态。但是，通过 transitions 可以在状态改变时播放动画。

例如，一个 button 被按下时，它的背景颜色可以在 300ms 的时间内动态的变为 perssed color。

transitions 的参数储存在样式中，它可以去设置：

- the time of the transition
- the delay before starting the transition 
- the animation path (also known as timing or easing function)
- the properties to animate 

可以为每个状态定义 transitions 属性。例如，在 default 状态设置 500ms transitions 时间，当对象变为默认状态时会应用 500ms 的 transitions 时间。

在 pressed 状态设置 100ms 的 transitions 时间将意味着在进入 pressed 状态时有 100ms 的 transitions 时间。

所以这个配置示例将导致快速进入 pressed 状态，缓慢回退到 default 状态。

描述一个 transitions ，需要初始化  `lv_transition_dsc_t` 变量并添加到样式中。

```c
/*Only its pointer is saved so must static, global or dynamically allocated */
static const lv_style_prop_t trans_props[] = {
											LV_STYLE_BG_OPA, LV_STYLE_BG_COLOR,
											0, /*End marker*/
};

static lv_style_transition_dsc_t trans1;
lv_style_transition_dsc_init(&trans1, trans_props, lv_anim_path_ease_out, duration_ms, delay_ms);

lv_style_set_transition(&style1, &trans1);
```

## 颜色过滤器
TODO


## 主题
主题是样式的集合，如果有一个活动的主题，LVGL 会将其应用到每一个创建的 widget 上。

这个将会给UI提供一个默认的外观，可以通过添加更多的样式来进行修改 

每个显示器可以有不同的主题。例如，你可以在 TFT 上使用彩色主题，在单色辅助显示器上使用单色主题显示。

为显示器设置主题，有 2 步需要做：

1. 初始化一个主题
2. 将初始化的主题分配给显示器

主题初始化函数拥有不同的参数。这个示例展示了如何设置 "default" 主题

```c
lv_theme_t * th = lv_theme_default_init(display,  /*Use the DPI, size, etc from this display*/ 
                                        LV_COLOR_PALETTE_BLUE, LV_COLOR_PALETTE_CYAN,   /*Primary and secondary palette*/
                                        false,    /*Light or dark mode*/ 
                                        &lv_font_montserrat_10, &lv_font_montserrat_14, &lv_font_montserrat_18); /*Small, normal, large fonts*/
                                        
lv_disp_set_theme(display, th); /*Assign the theme to the display*/
```

The themes can be enabled in `lv_conf.h`. If the default theme is enabled by `LV_USE_THEME_DEFAULT 1` LVGL automatically initializes and sets it when a display is created. 

在 `lv_conf.h` 中可以使能主题。如果默认的主题由 `LV_USE_DEFAULT 1` 启用，LVGL 在创建显示器时自动初始化这个主题并分配

### 拓展主题

Built-in themes can be extended. 
If a custom theme is created a parent theme can be selected. The parent theme's styles will be added before the custom theme's styles. 
Any number of themes can be chained this way. E.g. default theme -> custom theme -> dark theme.

`lv_theme_set_parent(new_theme, base_theme)` extends the `base_theme` with the `new_theme`.

There is an example for it below.

## Examples

```eval_rst

.. include:: ../../examples/styles/index.rst

```

## API
```eval_rst

.. doxygenfile:: lv_style.h
  :project: lvgl

.. doxygenfile:: lv_theme.h
  :project: lvgl

.. doxygenfile:: lv_obj_style_gen.h
  :project: lvgl
  
.. doxygenfile:: lv_style_gen.h
  :project: lvgl
  

```

