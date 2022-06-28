# TypeScript手册

## 一、TypeScript安装

### 1. npm安装TypeScript

使用国内镜像：

``` shell
$ npm config set registry https://registry.npmmirror.com
```

安装typescript

``` shell
$ npm install -g typescript
```

## 二、TypeScript编译

### 1. 单文件编译

``` shell
$ tsc xxx.ts
```

### 2. 多文件编译

``` shell
$ tsc
```

编译当前目录中所有的ts文件

### 3. tsconfig.json

使用ts的配置文件

``` shell
$ 
```



## 三、TypeScript的数据类型

### 1. 指定类型

``` typescript
let a: string;
```

### 2. 类型推断

``` typescript
let a = true; // a为boolean类型
```

### 3. 基础类型

- 任意类型：`any`
  ``` typescript
  let num1: number = 100; // 十进制数
  let num2: number = 0b10010; // 二进制数
  let num3: number = 0o1000; // 八进制数
  let num4: number = 0x1000; // 十六进制数
  
  let num5: number = 100.88;
  ```

  

- 数值类型：`number`

- 布尔类型：`boolean`

- 字符串类型：`string`

- 数组：
  ``` typescript
  // 普通数组
  let arr1:string[] = [];
  arr1.push("Alice");
  arr1.push("Bob");
  arr1.push("Candy");
  arr1.push("Daniel");
  arr1.push("Frank");
  
  console.log(arr1);
  
  // 泛型式数组声明
  let arr2:Array<string>=[];
  arr2.push("Alice");
  arr2.push("Bob");
  arr2.push("Candy");
  arr2.push("Daniel");
  arr2.push("Frank");
  
  console.log(arr2);
  
  ```

- 任意类型：`any`

- 未知类型：`unknown`，是一个类型安全的类型，不能将已有值类型的值，赋值给其它类型的变量；

- 空值：`void`，当函数无返回值时使用

- 永远：`never`，永远无返回值类型

- 未定义：`undefined`，未定义

### 函数

- 函数参数后缀为`?`表示此变量是可选的，可有可无，如：
  ``` typescript
  function add2(num1: number, num2: number, operator?: string): number {
    console.log("操作符是：" + operator || "无值");
    let value:number = 0;
    if (operator === undefined || operator === "+") {
      value = num1 + num2;
    } else if (operator == "-") {
      value = num1 - num2;
    } else if (operator == "*") {
      value = num1 * num2;
    } else if (operator == "/") {
      value = num1 / num2;
    } else {
      value = num1 + num2;
    }
  
    return value;
  }
  
  console.log(add2(23, 44, '-'));
  ```

  

- 参数个数不确定
  ``` typescript
  function showInfo: {name: string, age: number, [param: stri]}
  ```

  

- ``` ty
  ```

  
