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



A style on its own is not that useful, it needs to be assigned to an object to take effect.

### Add styles
To add a style to an object use `lv_obj_add_style(obj, &style, <selector>)`. `<selector>` is an OR-ed value of parts and state to which the style should be added. Some examples:
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

### Remove styles
To remove all styles from an object use `lv_obj_remove_style_all(obj)`.

To remove specific styles use `lv_obj_remove_style(obj, style, selector)`. This function will remove `style` only if the `selector` matches with the `selector` used in `lv_obj_add_style`. 
`style` can be `NULL` to check only the `selector` and remove all matching styles. The `selector` can use the `LV_STATE_ANY` and `LV_PART_ANY` values to remove the style with any state or part.


### Report style changes
If a style which is already assigned to object changes (i.e. a property is added or changed) the objects using that style should be notified. There are 3 options to do this:
1. If you know that the changed properties can be applied by a simple redraw (e.g. color or opacity changes) just call `lv_obj_invalidate(obj)` or `lv_obj_invalideate(lv_scr_act())`. 
2. If more complex style properties were changed or added, and you know which object(s) are affected by that style call `lv_obj_refresh_style(obj, part, property)`. 
To refresh all parts and properties use `lv_obj_refresh_style(obj, LV_PART_ANY, LV_STYLE_PROP_ANY)`.
3. To make LVGL check all objects to see whether they use the style and refresh them when needed call `lv_obj_report_style_change(&style)`. If `style` is `NULL` all objects will be notified about the style change.

### Get a property's value on an object
To get a final value of property - considering cascading, inheritance, local styles and transitions (see below) - get functions like this can be used: 
`lv_obj_get_style_<property_name>(obj, <part>)`. 
These functions use the object's current state and if no better candidate returns a default value.  
For example:
```c
lv_color_t color = lv_obj_get_style_bg_color(btn, LV_PART_MAIN);
```

## Local styles
Besides, "normal" styles, the objects can store local styles too. This concept is similar to inline styles in CSS (e.g. `<div style="color:red">`) with some modification. 

So local styles are like normal styles, but they can't be shared among other objects. If used, local styles are allocated automatically, and freed when the object is deleted.
They are useful to add local customization to the object.

Unlike in CSS, in LVGL local styles can be assigned to states (*pseudo-classes*) and parts (*pseudo-elements*).

To set a local property use functions like `lv_obj_set_style_local_<property_name>(obj, <value>, <selector>);`  
For example:
```c
lv_obj_set_style_local_bg_color(slider, lv_color_red(), LV_PART_INDICATOR | LV_STATE_FOCUSED);
```
## Properties
For the full list of style properties click [here](/overview/style-props).

### Typical background properties
In the documentation of the widgets you will see sentences like "The widget uses the typical background properties". The "typical background properties" are the ones related to:
- Background
- Border
- Outline
- Shadow
- Padding
- Width and height transformation
- X and Y translation


## Transitions
By default, when an object changes state (e.g. it's pressed) the new properties from the new state are set immediately. However, with transitions it's possible to play an animation on state change.
For example, on pressing a button its background color can be animated to the pressed color over 300 ms.

The parameters of the transitions are stored in the styles. It's possible to set 
- the time of the transition
- the delay before starting the transition 
- the animation path (also known as timing or easing function)
- the properties to animate 

The transition properties can be defined for each state. For example, setting 500 ms transition time in default state will mean that when the object goes to the default state a 500 ms transition time will be applied. 
Setting 100 ms transition time in the pressed state will mean a 100 ms transition time when going to pressed state.
So this example configuration will result in going to pressed state quickly and then going back to default slowly. 

To describe a transition an `lv_transition_dsc_t` variable needs to be initialized and added to a style:
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

## Color filter
TODO


## Themes
Themes are a collection of styles. If there is an active theme LVGL applies it on every created widget.
This will give a default appearance to the UI which can then be modified by adding further styles.

Every display can have a different theme. For example, you could have a colorful theme on a TFT and monochrome theme on a secondary monochrome display.

To set a theme for a display, 2 steps are required:
1. Initialize a theme
2. Assign the initialized theme to a display.

Theme initialization functions can have different prototype. This example shows how to set the "default" theme:
```c
lv_theme_t * th = lv_theme_default_init(display,  /*Use the DPI, size, etc from this display*/ 
                                        LV_COLOR_PALETTE_BLUE, LV_COLOR_PALETTE_CYAN,   /*Primary and secondary palette*/
                                        false,    /*Light or dark mode*/ 
                                        &lv_font_montserrat_10, &lv_font_montserrat_14, &lv_font_montserrat_18); /*Small, normal, large fonts*/
                                        
lv_disp_set_theme(display, th); /*Assign the theme to the display*/
```


The themes can be enabled in `lv_conf.h`. If the default theme is enabled by `LV_USE_THEME_DEFAULT 1` LVGL automatically initializes and sets it when a display is created. 

### Extending themes

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
