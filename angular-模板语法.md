1、使用插值表达式显示组件属性。`eg.<h1>{{title}}</h1>`

2、存放组件模板有两种形式。

* 内联模板。使用`template`属性定义
* 模板文件。使用`templateUrl`属性定义

3、初始化属性的两种方式
```
//变量赋值
export class AppCtorComponent {
  title: string;
  myHero: string;
  constructor() {
  //用构造函数来声明和初始化属性
    this.title = 'Tour of Heroes';
    this.myHero = 'Windstorm';
  }
}
```

4、*ngIf*会根据布尔条件来显示或移除一个元素。它并不是在显示和隐藏这条消息，它是从DOM中添加和移除这个段落元素。这会提高性能。

5、表达式应该遵循以下指南

* 没有可见的副作用。即除了目标属性的值外，不应该改变应用的任何状态。
* 执行迅速。
* 非常简单。把应用程序和业务逻辑限制在组件中

6、模板语句
模板语句出现在=号右侧的引号中。
```
<button (click)="deleteHero()">Delete hero</button>
```

7、语句上下文可以引用模板自身上下文中的属性。
```
//传模板的$event对象
<button (click)="onSave($event)">Save</button>
//传模板输入变量
<button *ngFor="let hero of heroes" (click)="deleteHero(hero)">{{hero.name}}</button>
//传模板引用变量
<form #heroForm (ngSubmit)="onSubmit(heroForm)"> ... </form>
```

8、数据绑定

* （单向）从数据源到视图
```
{{expression}}
[target]="expression"
bind-target="expression"
```
* （单向）从视图到数据源
```
(target)="statement"
on-target="statement"
```
* （双向）从视图到数据源再到视图
```
[(target)]="expression"
bindon-target="expression"
```

9、`HTML attribute`与`DOM property`的对比。

* `attribute`初始化`DOM property`，然后它们的任务就完成了。`property`的值可以改变；`attribute`的值不能改变。
* **模板绑定是通过property和事件来工作的，而不是attribute。**
```
//这里不是设置attribute，而是设置DOM元素、组件和指令的property
<button [disabled]="isUnchanged">Save</button>
```

10、绑定目标。这个目标可能是（元素、组件、指令）的property、（元素、组件、指令）的事件，或（极少数情况下）attribute名。
```
//property
<img [src]="heroImageUrl">
<app-hero-detail [hero]="currentHero"></app-hero-detail>
<div [ngClass]="{'special': isSpecial}"></div>
//事件
<button (click)="onSave()">Save</button>
<app-hero-detail (deleteRequest)="deleteHero()"></app-hero-detail>
<div (myClick)="clicked=$event" clickable>click me</div>
//双向
<input [(ngModel)]="name">
//attribute
<button [attr.aria-label]="help">help</button>
//CSS
<div [class.special]="isSpecial">Special</div>
//样式
<button [style.color]="isSpecial ? 'red' : 'green'">
```

11、一次性字符串初始化

* 目标属性接收字符串值
* 字符串是个固定值，可以直接合并到模块中
* 这个初始值永不改变
当满足以上条件时，应该省略括号
```
//给组件的prefix属性初始化为固定的字符串，而不是模板表达式
<app-hero-detail prefix="You are my" [hero]="currentHero"></app-hero-detail>
```

12、属性绑定和插值
```
//两种写法是等价的
<p><img src="{{heroImageUrl}}"> is the <i>interpolated</i> image.</p>
<p><img [src]="heroImageUrl"> is the <i>property bound</i> image.</p>

<p><span>"{{title}}" is the <i>interpolated</i> title.</span></p>
<p>"<span [innerHTML]="title"></span>" is the <i>property bound</i> title.</p>
```

13、`attribute`绑定

* 当元素没有属性(property)可以绑定的时候，就必须使用`attribute`绑定。
* 绑定方式：由attr前缀，一个点(.)和attribute的名字组成。`<tr><td [attr.colspan]="1 + 1">One-Two</td></tr>`。
* attribute绑定的主要用例之一是设置ARIA attribute（给残障人士使用）

14、css类绑定:由class前缀，一个点(.)和css 类的名字组成，其中后两部分是可选的。
```
<!-- toggle the "special" class on/off with a property -->
<div [class.special]="isSpecial">The class binding is special</div>

<!-- binding to `class.special` trumps the class attribute -->
<div class="special"
     [class.special]="!isSpecial">This one is not so special</div>
```

15、样式绑定：由style前缀，一个点(.)和css样式的属性名组成。
```
<button [style.color]="isSpecial ? 'red': 'green'">Red</button>
<button [style.background-color]="canSave ? 'cyan': 'grey'" >Save</button>
//样式绑定中样式带有单位，根据条件用em和%来设置
<button [style.font-size.em]="isSpecial ? 3 : 1" >Big</button>
<button [style.fontSize.%]="!isSpecial ? 150 : 50" >Small</button>
```

16、内置属性型指令：会监听和修改其他HTML元素或组件的行为、元素属性(attribute)、DOM属性(property)。它们通常作为HTML属性的名称而应用在元素上。

*  ngClass
```
//添加或删除单个类的最佳途径
<div [class.special]="isSpecial">The class binding is special</div>
//同时添加或移除多个类
currentClasses: {};
setCurrentClasses() {
  // CSS classes: added/removed per current state of component properties
  this.currentClasses =  {
    'saveable': this.canSave,
    'modified': !this.isUnchanged,
    'special':  this.isSpecial
  };
}
//根据属性的状态为true或false而添加或移除对应的类
<div [ngClass]="currentClasses">This div is initially saveable, unchanged, and special</div>
```
* ngStyle
```
//样式绑定是设置单一样式值的简单方式
<div [style.font-size]="isSpecial ? 'x-large' : 'smaller'" >
  This div is x-large or smaller.
</div>
//同时设置多个内联样式，用ngStyle指令
currentStyles: {};
setCurrentStyles() {
  // CSS styles: set per current state of component properties
  this.currentStyles = {
    'font-style':  this.canSave      ? 'italic' : 'normal',
    'font-weight': !this.isUnchanged ? 'bold'   : 'normal',
    'font-size':   this.isSpecial    ? '24px'   : '12px'
  };
}
<div [ngStyle]="currentStyles">
  This div is initially italic, normal weight, and extra large (24px).
</div>
```
* ngModel
    * 使用ngModel双向绑定之前，必须导入FormsModule并把它添加到NgModule的imports列表中。
    * ngModel指令 **不能** 用到非表单类的原生元素或第三方自定义组件上，除非写一个合适的值访问器。
```
<input [(ngModel)]="currentHero.name">
//等价于如下拆分
<input
  [ngModel]="currentHero.name"
  (ngModelChange)="setUppercaseName($event)">
```

17、内置结构型指令：通过添加、移除宿主元素来修改DOM结构。

* 当没有合适的宿主元素放置指令是，可用`<ng-container>`对元素进行分组
* 一个元素只能应用一个结构型指令
* ngIf/ngFor

18、带`trackBy`的ngFor

```
//有了 trackBy，则只有修改了 id 的按钮才会触发元素替换。类似于react中的key
trackByHeroes(index: number, hero: Hero): number { return hero.id; }

<div *ngFor="let hero of heroes; trackBy: trackByHeroes">
  ({{hero.id}}) {{hero.name}}
</div>
```
