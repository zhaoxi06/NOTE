1、通过$event对象获取用户输入
```
@Component({
  selector: 'app-click-me',
  template: `
    <input (keyup)="onKey($event)">
    <p>{{values}}</p>`
})
export class KeyUpComponent_v1 {
  values = '';
  onKey(event: any) { // without type info
    this.values += event.target.value + ' | ';
  }
}
```
把$event对象传到方法中，组件会知道太多模板的信息，这就违背了模板和组件之间分离的原则。

2、模板引用变获取用户输入
```
@Component({
  selector: 'app-loop-back',
  template: `
  //必须绑定一个事件，虽然这个事件不做什么 (keyup)="0"
    <input #box (keyup)="0">
    <p>{{box.value}}</p>
  `
})
export class LoopbackComponent { }

//优化
<input #box (keyup)="onKey(box.value)">
<p>{{values}}</p>
```

3、按键事件过滤(key.enter)<br>
keyup.enter:只有当用户敲回车键时，才会调用事件
```
<input #box (keyup.enter)="onEnter(box.value)">
<p>{{value}}</p>
//如果用户没有先按回车键，而是移开了鼠标，点击了页面中其它地方，输入框的当前值就会丢失。 只有当用户按下了回车键候，组件的 values 属性才能更新.
```

4、失去焦点事件
```
//同时监听输入框的回车键和失去焦点事件来修正上述问题
<input #box (keyup.enter)="update(box.value)"
            (blur)="update(box.value)">
<p>{{value}}</p>
```
