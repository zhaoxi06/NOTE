# 一、基础类型
1、使用模板字符串，定义多行文本和内嵌表达式。这种字符串是被反引号包围（ `），并且以${ expr }这种形式嵌入表达式。<br>
```
let name: string = `Gene`;
let age: number = 37;
let sentence: string = `Hello, my name is ${ name }.

I'll be ${ age + 1 } years old next month.`;
```
2、定义数组有以下两种方式<br>
```
let list: number[] = [1, 2, 3];
let list: Array<number> = [1, 2, 3];
```
3、元组`Tuple`类型允许表示一个已知元素数量和类型的数组，各元素的类型不必相同。<br>
```
// Declare a tuple type
let x: [string, number];
// Initialize it
x = ['hello', 10]; // OK
// Initialize it incorrectly
x = [10, 'hello']; // Error
//当访问一个元组中越界的元素，会使用联合类型替代。在这个例子中，联合类型是(string | number)
x[3] = 'world'; // OK, 字符串可以赋值给(string | number)类型
console.log(x[5].toString()); // OK, 'string' 和 'number' 都有 toString
x[6] = true; // Error, 布尔不是(string | number)类型
```
4、枚举`enum`<br>
```
//默认情况下，从0开始为元素编号
enum Color {Red, Green, Blue}
let c: Color = Color.Green;
console.log(c);     //1
//也可以手动赋值
enum Color {Red = 1, Green, Blue}
//可以由枚举的值得到它的名字
let colorName: string = Color[2];
console.log(colorName);  // 显示'Green'因为上面代码里它的值是2
```
5、`Any`为不清楚类型的变量指定一个类型，使得它们通过编译阶段的检查。或者定义一个数组，它包含了不同类型的数据。
```
let list: any[] = [1, true, "free"];
list[1] = 100;
```
6、`void` 表示没有任何类型。当一个函数没有返回值时，它的返回值类型就是void。只能为它赋值为`undefined`和`null`。
```
function warnUser(): void {
    console.log("This is my warning message");
}
let unusable: void = undefined;
```
7、默认情况下`null`和`undefined`是所有类型的子类型。 就是说你可以把 `null`和`undefined`赋值给`number`类型的变量。然而，当你指定了--strictNullChecks标记，null和undefined只能赋值给void和它们各自。<br>
8、`never`类型表示的是那些永不存在的值的类型。 例如， never类型是那些总是会抛出异常或根本就不会有返回值的函数表达式或箭头函数表达式的返回值类型； 变量也可能是 never类型，当它们被永不为真的类型保护所约束时。
never类型是任何类型的子类型，也可以赋值给任何类型；然而，没有类型是never的子类型或可以赋值给never类型（除了never本身之外）。 *即使 any也不可以赋值给never。*<br>
```
// 返回never的函数必须存在无法达到的终点
function error(message: string): never {
    throw new Error(message);
}
// 推断的返回值类型为never
function fail() {
    return error("Something failed");
}
// 返回never的函数必须存在无法达到的终点
function infiniteLoop(): never {
    while (true) {
    }
}
```
8、`object`表示非原始类型，也就是除number，string，boolean，symbol，null或undefined之外的类型。<br>
9、类型断言。有时候你会遇到这样的情况，你会比TypeScript更了解某个值的详细信息。 通常这会发生在你清楚地知道一个实体具有比它现有类型更确切的类型。
通过类型断言这种方式可以告诉编译器，“相信我，我知道自己在干什么”。 类型断言好比其它语言里的类型转换，但是不进行特殊的数据检查和解构。 它没有运行时的影响，只是在编译阶段起作用。类型断言有两种形式。
```
//其一是“尖括号”语法
let someValue: any = "this is a string";
let strLength: number = (<string>someValue).length;
//另一个为as语法
let someValue: any = "this is a string";
let strLength: number = (someValue as string).length;
```
两种形式是等价的。*当你在TypeScript里使用JSX时，只有 as语法断言是被允许的。*

# 二、接口
1、使用接口来描述：必须包含一个label属性且类型为string<br>
```
interface LabelledValue {
  label: string;
}
function printLabel(labelledObj: LabelledValue) {
  console.log(labelledObj.label);
}
let myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```
2、可选属性<br>
```
interface SquareConfig {
  color?: string;
  width?: number;
}
```
3、只读属性<br>
```
interface Point {
    readonly x: number;
    readonly y: number;
}
let a: number[] = [1, 2, 3, 4];
let ro: ReadonlyArray<number> = a;
ro[0] = 12; // error!
a = ro; // error!
//可以用类型断言重写
a = ro as number[];
```
4、`readonly` vs `const`。最简单判断该用readonly还是const的方法是看要把它做为变量使用还是做为一个属性。 做为变量使用的话用 const，若做为属性则使用readonly。<br>
5、SquareConfig可以有任意数量的属性，并且只要它们不是color和width，那么就无所谓它们的类型是什么。<br>
```
interface SquareConfig {
    color?: string;
    width?: number;
}
function createSquare(config: SquareConfig): { color: string; area: number } {
    // ...
}
// error: 'colour' not expected in type 'SquareConfig'
let mySquare = createSquare({ colour: "red", width: 100 });
```
用以下方法绕开不必要的检查<br>
```
//第一种：使用类型断言
let mySquare = createSquare({ width: 100, opacity: 0.5 } as SquareConfig);
//第二种：添加一个字符串索引签名
interface SquareConfig {
    color?: string;
    width?: number;
    [propName: string]: any;
}
//第三种：将这个对象赋值给一个另一个变量
let squareOptions = { colour: "red", width: 100 };
let mySquare = createSquare(squareOptions);
```
6、接口也可以描述函数类型。函数的参数会逐个进行检查，要求对应位置上的参数类型是兼容的。<br>
```
interface SearchFunc {
  (source: string, subString: string): boolean;
}
let mySearch: SearchFunc;
mySearch = function(source: string, subString: string) {
  let result = source.search(subString);
  return result > -1;
}
```
