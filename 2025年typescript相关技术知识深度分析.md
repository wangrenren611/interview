# 2025年TypeScript相关技术知识深度分析

## 前言

TypeScript作为JavaScript的超集，在2025年继续引领着前端开发的发展趋势。随着TypeScript 5.5、5.6以及即将到来的TypeScript 7.0（原生Go版本）的发布，TypeScript在性能、类型系统和开发体验方面都有了显著提升。本文将深入分析TypeScript的核心概念、高级特性、最佳实践以及2025年的最新发展趋势。

## 一、TypeScript基础概念与优势

### 1.1 什么是TypeScript

TypeScript是Microsoft开发的开源编程语言，它是JavaScript的一个超集，添加了可选的静态类型和基于类的面向对象编程。TypeScript可以编译成纯JavaScript，并且可以在任何浏览器、Node.js或任何支持ECMAScript 3或更高版本的JavaScript引擎中运行。

### 1.2 TypeScript的主要优势

1. **静态类型检查**：在编译时发现错误，而不是在运行时
2. **更好的IDE支持**：智能提示、重构和导航功能
3. **代码可维护性**：明确的类型定义使代码更易于理解和维护
4. **团队协作**：类型定义作为文档，减少沟通成本
5. **渐进式采用**：可以逐步将JavaScript项目迁移到TypeScript

### 1.3 TypeScript与JavaScript的关系

```
// 这段代码是有效的JavaScript，也是有效的TypeScript
function greet(name) {
  return "Hello, " + name;
}

// 添加类型注解后，这就是TypeScript特有的写法
function greetTyped(name: string): string {
  return "Hello, " + name;
}
```

## 二、TypeScript类型系统详解

### 2.1 基础类型

TypeScript包含JavaScript的所有基本类型，同时还引入了额外的类型：

```
// 基本类型
let isDone: boolean = false;
let count: number = 42;
let userName: string = "TypeScript";
let list: number[] = [1, 2, 3];
let tuple: [string, number] = ["hello", 10];

// 枚举
enum Color { Red, Green, Blue }
let c: Color = Color.Green;

// Any和Unknown
let notSure: any = 4;
notSure = "maybe a string instead";

// Unknown比Any更安全
let value: unknown = 4;
// value.toFixed(); // 错误：Object is of type 'unknown'
if (typeof value === 'number') {
  value.toFixed(); // 正确
}
```

### 2.2 接口与类型别名

```
// 接口定义对象结构
interface Person {
  name: string;
  age: number;
  greet(): void;
}

// 可选属性和只读属性
interface SquareConfig {
  color?: string;
  readonly width: number;
}

// 类型别名
type Point = {
  x: number;
  y: number;
};

// 联合类型
type Status = "loading" | "success" | "error";

// 交叉类型
type Named = { name: string };
type Aged = { age: number };
type NamedAndAged = Named & Aged;
```

### 2.3 泛型

泛型允许我们创建可重用的组件，一个组件可以支持多种类型的数据：

```
// 简单泛型函数
function identity<T>(arg: T): T {
  return arg;
}

// 使用泛型
let output = identity<string>("myString");
let output2 = identity("myString"); // 类型推断

// 泛型接口
interface GenericIdentityFn<T> {
  (arg: T): T;
}

// 泛型类
class GenericNumber<T> {
  zeroValue: T;
  add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };
```

## 三、TypeScript高级类型操作

### 3.1 条件类型

条件类型允许我们根据类型关系来选择不同的类型：

```
// 基本条件类型
type MessageOf<T> = T extends { message: unknown } ? T["message"] : never;

interface Email {
  message: string;
}

interface Dog {
  bark(): void;
}

type EmailMessageContents = MessageOf<Email>; // string
type DogMessageContents = MessageOf<Dog>; // never

// 使用infer关键字
type Flatten<Type> = Type extends Array<infer Item> ? Item : Type;

type Str = Flatten<string[]>; // string
type Num = Flatten<number>; // number

// 获取函数返回类型
type GetReturnType<Type> = Type extends (...args: never[]) => infer Return ? Return : never;

type NumType = GetReturnType<() => number>; // number
```

### 3.2 映射类型

映射类型通过在现有类型的基础上创建新类型：

```
// 基本映射类型
type OptionsFlags<Type> = {
  [Property in keyof Type]: boolean;
};

type Features = {
  darkMode: () => void;
  newUserProfile: () => void;
};

type FeatureOptions = OptionsFlags<Features>;
// {
//   darkMode: boolean;
//   newUserProfile: boolean;
// }

// 映射修饰符
type CreateMutable<Type> = {
  -readonly [Property in keyof Type]: Type[Property];
};

type LockedAccount = {
  readonly id: string;
  readonly name: string;
};

type UnlockedAccount = CreateMutable<LockedAccount>;
// {
//   id: string;
//   name: string;
// }

// 使用as进行键名重映射
type Getters<Type> = {
  [Property in keyof Type as `get${Capitalize<string & Property>}`]: () => Type[Property]
};

interface Person {
  name: string;
  age: number;
}

type LazyPerson = Getters<Person>;
// {
//   getName: () => string;
//   getAge: () => number;
// }
```

### 3.3 模板字面量类型

模板字面量类型基于字符串字面量类型构建，并可以通过联合类型扩展为多个字符串：

```
type World = "world";
type Greeting = `hello ${World}`; // "hello world"

type EmailLocaleIDs = "welcome_email" | "email_heading";
type FooterLocaleIDs = "footer_title" | "footer_sendoff";

type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`;
// "welcome_email_id" | "email_heading_id" | "footer_title_id" | "footer_sendoff_id"

// 内置字符串操作类型
type Upper = Uppercase<'hello'>; // "HELLO"
type Lower = Lowercase<'HELLO'>; // "hello"
type Capital = Capitalize<'hello'>; // "Hello"
type Uncapital = Uncapitalize<'Hello'>; // "hello"
```

## 四、TypeScript 5.5新特性详解

### 4.1 推断类型谓词

TypeScript 5.5改进了类型谓词的推断能力，使得在数组过滤操作中能够自动推断更精确的类型：

```
interface Bird {
  commonName: string;
  scientificName: string;
  sing(): void;
}

// Maps country names -> national bird.
declare const nationalBirds: Map<string, Bird>;

function makeBirdCalls(countries: string[]) {
  // 在TypeScript 5.5之前：birds: (Bird | undefined)[]
  // 在TypeScript 5.5之后：birds: Bird[]
  const birds = countries
    .map(country => nationalBirds.get(country))
    .filter(bird => bird !== undefined);

  for (const bird of birds) {
    bird.sing(); // OK!
  }
}

// 更多推断类型谓词的例子
const isNumber = (x: unknown) => typeof x === 'number';
// 推断为: (x: unknown) => x is number

const isNonNullish = <T,>(x: T) => x != null;
// 推断为: <T>(x: T) => x is NonNullable<T>
```

### 4.2 常量索引访问的控制流收窄

TypeScript现在能够收窄形式为obj[key]的表达式，当obj和key都是有效的常量时：

```
function f1(obj: Record<string, unknown>, key: string) {
  if (typeof obj[key] === "string") {
    // 现在可以了，之前会报错
    obj[key].toUpperCase();
  }
}
```

### 4.3 JSDoc @import标签

在JavaScript文件中，现在可以使用@import标签来导入仅用于类型检查的内容：

```
// ./some-module.d.ts
export interface SomeType {
  // ...
}

// ./index.js
/** @import { SomeType } from "some-module" */

/**
 * @param {SomeType} myValue
 */
function doSomething(myValue) {
  // ...
}
```

### 4.4 正则表达式语法检查

TypeScript现在对正则表达式进行基本的语法检查：

```
// 错误：意外的')'，你是否想用反斜杠转义？
let myRegex = /@robot(\s+(please|immediately)))? do some task/;
//                                            ~

// 错误：这个反向引用指向一个不存在的组
let myRegex2 = /@typedef \{import\((.+)\)\.([a-zA-Z_]+)\} \3/u;
//                                                        ~
```

## 五、TypeScript装饰器与元数据反射

### 5.1 装饰器基础

装饰器是一种特殊类型的声明，可以被附加到类声明、方法、访问符、属性或参数上。装饰器使用@expression这种形式，expression求值后必须为一个函数，它会在运行时被调用。

```
// 类装饰器
function sealed(constructor: Function) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}

@sealed
class BugReport {
  type = "report";
  title: string;

  constructor(t: string) {
    this.title = t;
  }
}

// 方法装饰器
function enumerable(value: boolean) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    descriptor.enumerable = value;
  };
}

class Greeter {
  greeting: string;
  constructor(message: string) {
    this.greeting = message;
  }

  @enumerable(false)
  greet() {
    return "Hello, " + this.greeting;
  }
}
```

### 5.2 TypeScript 5.0+装饰器

TypeScript 5.0引入了新的装饰器语法，更加符合ECMAScript装饰器提案：

```
// TypeScript 5.0+装饰器示例
function loggedMethod(originalMethod: any, context: ClassMethodDecoratorContext) {
  const methodName = String(context.name);
  
  function replacementMethod(this: any, ...args: any[]) {
    console.log(`LOG: Entering method '${methodName}'.`)
    const result = originalMethod.call(this, ...args);
    console.log(`LOG: Exiting method '${methodName}'.`)
    return result;
  }
  
  return replacementMethod;
}

class Person {
  name: string;
  constructor(name: string) {
    this.name = name;
  }

  @loggedMethod
  greet() {
    console.log(`Hello, my name is ${this.name}.`);
  }
}
```

### 5.3 元数据反射

使用reflect-metadata库可以实现装饰器的元数据反射功能：

```
// 需要安装: npm install reflect-metadata
// 需要在项目中导入一次: import "reflect-metadata"

import "reflect-metadata";

function MinLength(minLength: number) {
  return function (target: any, propertyKey: string) {
    // 存储元数据
    Reflect.defineMetadata("minLength", minLength, target, propertyKey);
  };
}

class User {
  @MinLength(5)
  password: string;
  
  constructor(password: string) {
    this.password = password;
  }
}

// 读取元数据
const minLength = Reflect.getMetadata("minLength", User.prototype, "password");
console.log(minLength); // 5
```

## 六、TypeScript性能优化与最佳实践

### 6.1 TypeScript 7.0（原生Go版本）性能提升

Microsoft正在将TypeScript编译器从JavaScript移植到Go语言，预计性能将提升10倍：

```
# 性能对比（早期测试数据）
# Codebase     | Size (LOC) | Current | Native | Speedup
# VS Code      | 1,505,000  | 77.8s   | 7.5s   | 10.4x
# Playwright   | 356,000    | 11.1s   | 1.1s   | 10.1x
# TypeORM      | 270,000    | 17.5s   | 1.3s   | 13.5x
```

### 6.2 类型安全最佳实践

```
// 1. 避免使用any，优先使用unknown
// 不好的做法
function processValue(value: any) {
  return value.toFixed(2); // 运行时可能出错
}

// 好的做法
function processValueSafe(value: unknown) {
  if (typeof value === 'number') {
    return value.toFixed(2);
  }
  throw new Error('Value is not a number');
}

// 2. 使用satisfies操作符
type Theme = 'light' | 'dark';
const themeConfig = {
  light: { bgColor: '#fff', textColor: '#000' },
  dark: { bgColor: '#000', textColor: '#fff' }
} satisfies Record<Theme, { bgColor: string; textColor: string }>;

// 3. 合理使用类型守卫
function isString(value: unknown): value is string {
  return typeof value === 'string';
}

// 4. 使用const断言
const colors = {
  red: '#ff0000',
  green: '#00ff00',
  blue: '#0000ff'
} as const;

type Color = keyof typeof colors; // "red" | "green" | "blue"
```

### 6.3 模块和命名空间最佳实践

```
// 1. 使用ES模块而不是命名空间
// 不推荐
namespace Validation {
  export interface StringValidator {
    isAcceptable(s: string): boolean;
  }
}

// 推荐
export interface StringValidator {
  isAcceptable(s: string): boolean;
}

// 2. 合理组织模块结构
// validators/index.ts
export * from './stringValidator';
export * from './numberValidator';

// validators/stringValidator.ts
export interface StringValidator {
  isAcceptable(s: string): boolean;
}

export class ZipCodeValidator implements StringValidator {
  isAcceptable(s: string) {
    return s.length === 5 && /^\d+$/.test(s);
  }
}
```

## 七、TypeScript与现代前端框架集成

### 7.1 React与TypeScript

```
// 函数组件
interface ButtonProps {
  children: React.ReactNode;
  variant?: 'primary' | 'secondary';
  onClick: () => void;
}

const Button: React.FC<ButtonProps> = ({ 
  children, 
  variant = 'primary', 
  onClick 
}) => {
  return (
    <button 
      className={`btn btn-${variant}`} 
      onClick={onClick}
    >
      {children}
    </button>
  );
};

// 自定义Hook
interface UseApiResult<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
  refetch: () => void;
}

function useApi<T>(url: string): UseApiResult<T> {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<Error | null>(null);

  const fetchData = async () => {
    try {
      setLoading(true);
      const response = await fetch(url);
      const result = await response.json();
      setData(result);
    } catch (err) {
      setError(err instanceof Error ? err : new Error('Unknown error'));
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchData();
  }, [url]);

  return { data, loading, error, refetch: fetchData };
}
```

### 7.2 Vue与TypeScript

```
// Vue 3 Composition API with TypeScript
import { defineComponent, ref, computed, PropType } from 'vue';

interface User {
  id: number;
  name: string;
  email: string;
}

export default defineComponent({
  name: 'UserCard',
  props: {
    user: {
      type: Object as PropType<User>,
      required: true
    },
    editable: {
      type: Boolean,
      default: false
    }
  },
  emits: {
    'update:user': (user: User) => true
  },
  setup(props, { emit }) {
    const localUser = ref<User>({ ...props.user });
    
    const fullName = computed(() => {
      return `${localUser.value.name}`;
    });

    const updateUser = () => {
      emit('update:user', localUser.value);
    };

    return {
      localUser,
      fullName,
      updateUser
    };
  }
});
```

## 八、2025年TypeScript发展趋势

### 8.1 原生编译器性能革命

随着TypeScript 7.0（原生Go版本）的开发，我们可以期待：

1. 编辑器启动速度提升10倍
2. 构建时间减少10倍
3. 内存使用量大幅降低
4. 更好的AI工具集成支持

### 8.2 更强大的类型系统

1. 更智能的类型推断
2. 更丰富的条件类型操作
3. 更好的模板字面量类型支持
4. 改进的装饰器和元编程能力

### 8.3 与新兴技术的集成

1. 更好的WebAssembly支持
2. 增强的Web Components类型支持
3. 与AI开发工具的深度集成
4. 对新兴JavaScript特性的更快支持

## 结语

TypeScript在2025年继续巩固其在前端开发领域的地位。随着性能的大幅提升、类型系统的不断完善以及与现代前端框架的深度集成，TypeScript为开发者提供了更强大、更安全、更高效的开发体验。掌握TypeScript的核心概念和高级特性，将有助于开发者构建更高质量的现代Web应用。

通过本文的深度分析，我们了解了TypeScript的基础概念、类型系统、高级特性、新版本特性以及最佳实践。希望这些内容能够帮助开发者更好地使用TypeScript，构建出更加健壮和可维护的应用程序。

## 九、TypeScript面试题与答案解析

### 9.1 初级面试题

#### 1. 什么是TypeScript？它与JavaScript有什么区别？

**答案解析**：
TypeScript是Microsoft开发的开源编程语言，它是JavaScript的一个超集，添加了可选的静态类型和基于类的面向对象编程。主要区别包括：

1. **类型系统**：TypeScript支持静态类型检查，而JavaScript是动态类型
2. **编译时检查**：TypeScript在编译时发现错误，JavaScript在运行时才发现
3. **额外特性**：TypeScript提供了接口、泛型、枚举等JavaScript没有的特性
4. **编译过程**：TypeScript必须先编译成JavaScript才能在浏览器中运行

```
// JavaScript - 动态类型
let value = "Hello";
value = 42; // 完全合法

// TypeScript - 静态类型
let value: string = "Hello";
value = 42; // 编译错误: Type 'number' is not assignable to type 'string'
```

#### 2. TypeScript中的any、unknown和never类型有什么区别？

**答案解析**：
这三种类型在TypeScript中都有特殊用途，但安全性和使用场景不同：

1. **any类型**：最灵活的类型，允许赋值任何值，会跳过所有类型检查（不推荐使用）
2. **unknown类型**：比any更安全，表示未知类型，在使用前必须进行类型检查
3. **never类型**：表示永远不会发生的值，常用于函数永远不返回或总是抛出异常的情况

```
// any类型 - 跳过所有类型检查
let anyValue: any = "Hello";
anyValue.toFixed(); // 不会报错，但运行时可能出错

// unknown类型 - 更安全
let unknownValue: unknown = "Hello";
// unknownValue.toFixed(); // 编译错误
if (typeof unknownValue === 'number') {
  unknownValue.toFixed(); // 正确，类型检查后可以使用
}

// never类型 - 永不返回的函数
function throwError(message: string): never {
  throw new Error(message);
}

function infiniteLoop(): never {
  while (true) {}
}
```

#### 3. interface和type有什么区别？

**答案解析**：
interface和type都可以用来定义对象的结构，但它们有重要区别：

1. **扩展性**：interface可以通过声明合并来扩展，type不能
2. **用途**：interface主要用于定义对象形状，type可以为任何类型创建别名
3. **联合类型**：type可以更方便地处理联合类型和交叉类型

```typepescript
// interface可以声明合并
interface Person {
  name: string;
}

interface Person {
  age: number;
}

// 结果是: interface Person { name: string; age: number; }

// type不能声明合并
type PersonType = { name: string; };
// type PersonType = { age: number; }; // 错误：标识符重复

// type更适合联合类型
type Status = "loading" | "success" | "error";
type Result = { data: string } | { error: string };
```

### 9.2 中级面试题

#### 1. 什么是泛型？请举例说明其用途

**答案解析**：
泛型允许我们创建可重用的组件，一个组件可以支持多种类型的数据，同时保持类型安全。泛型提供了一种将类型作为参数传递给函数、类或接口的方式。

泛型的主要优势：
1. **类型安全**：在编译时确保类型正确
2. **代码复用**：同一个函数可以处理多种类型
3. **避免类型断言**：不需要使用any或强制类型转换

```typepescript
// 简单泛型函数
function identity<T>(arg: T): T {
  return arg;
}

// 使用
let output1 = identity<string>("myString"); // string类型
let output2 = identity<number>(42); // number类型
let output3 = identity("myString"); // 类型推断为string

// 泛型约束 - 限制泛型必须具有某些属性
interface Lengthwise {
  length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
  console.log(arg.length); // 现在可以安全访问length属性
  return arg;
}

loggingIdentity("hello"); // string有length属性
loggingIdentity([1, 2, 3]); // array有length属性
// loggingIdentity(3); // 错误：number没有length属性

// 泛型类
class GenericNumber<T> {
  zeroValue: T;
  add: (x: T, y: T) => T;
  
  constructor(zeroValue: T, add: (x: T, y: T) => T) {
    this.zeroValue = zeroValue;
    this.add = add;
  }
}

let myGenericNumber = new GenericNumber<number>(0, (x, y) => x + y);
console.log(myGenericNumber.add(10, 20)); // 30
```

#### 2. 什么是联合类型和交叉类型？如何使用？

**答案解析**：
联合类型和交叉类型是TypeScript中强大的类型组合方式：

1. **联合类型（Union Types）**：表示一个值可以是几种类型之一，使用`|`符号
2. **交叉类型（Intersection Types）**：将多个类型合并为一个类型，使用`&`符号

```typescript

// 联合类型
let value: string | number;
value = "hello"; // 正确
value = 42; // 正确
// value = true; // 错误

// 使用联合类型时需要类型守卫
function padLeft(value: string, padding: string | number) {
  if (typeof padding === "number") {
    return " ".repeat(padding) + value;
  }
  if (typeof padding === "string") {
    return padding + value;
  }
  throw new Error(`Expected string or number, got '${padding}'.`);
}

// 交叉类型
interface Person {
  name: string;
  age: number;
}

interface Employee {
  employeeId: number;
  department: string;
}

type EmployeePerson = Person & Employee;

const employee: EmployeePerson = {
  name: "John",
  age: 30,
  employeeId: 12345,
  department: "Engineering"
};

// 接口合并实现类似交叉类型的效果
interface Identifiable {
  id: string;
}

interface Timestamped {
  createdAt: Date;
}

interface User extends Identifiable, Timestamped {
  name: string;
}
```

#### 3. TypeScript中的装饰器是什么？如何使用？

**答案解析**：
装饰器是一种特殊类型的声明，可以被附加到类声明、方法、访问符、属性或参数上。装饰器使用@expression这种形式，expression求值后必须为一个函数，它会在运行时被调用。

装饰器主要用于：
1. **元编程**：在编译时修改类的行为
2. **框架开发**：如Angular大量使用装饰器
3. **AOP（面向切面编程）**：添加日志、权限检查等功能

```
// 需要在tsconfig.json中启用experimentalDecorators
// {
//   "compilerOptions": {
//     "experimentalDecorators": true
//   }
// }

// 类装饰器
function sealed(constructor: Function) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}

@sealed
class BugReport {
  type = "report";
  title: string;
  
  constructor(t: string) {
    this.title = t;
  }
}

// 方法装饰器
function enumerable(value: boolean) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    descriptor.enumerable = value;
  };
}

class Greeter {
  greeting: string;
  
  constructor(message: string) {
    this.greeting = message;
  }
  
  @enumerable(false)
  greet() {
    return "Hello, " + this.greeting;
  }
}

// 属性装饰器
function format(formatString: string) {
  return function (target: any, propertyKey: string) {
    // 存储格式信息到元数据
    Reflect.defineMetadata("format", formatString, target, propertyKey);
  };
}

class Person {
  @format("yyyy-MM-dd")
  birthDate: Date;
  
  constructor(birthDate: Date) {
    this.birthDate = birthDate;
  }
}

// 参数装饰器
function required(target: Object, propertyKey: string | symbol, parameterIndex: number) {
  // 标记参数为必需
  let existingRequiredParameters: number[] = Reflect.getMetadata("requiredParameters", target, propertyKey) || [];
  existingRequiredParameters.push(parameterIndex);
  Reflect.defineMetadata("requiredParameters", existingRequiredParameters, target, propertyKey);
}

class User {
  login(@required username: string, @required password: string) {
    // 登录逻辑
  }
}
```

### 9.3 高级面试题

#### 1. TypeScript如何处理异步编程？请举例说明Promise和async/await的类型安全

**答案解析**：
TypeScript为异步编程提供了完整的类型安全支持，包括Promise类型、async/await语法和错误处理。

关键概念：
1. **Promise类型**：Promise<T>表示一个最终会解析为T类型的值
2. **async函数**：自动返回Promise<T>类型
3. **await操作符**：解包Promise的值并保持类型安全
4. **错误处理**：通过类型系统确保错误被正确处理

```
// Promise类型安全
function fetchData(): Promise<string> {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve("Data fetched successfully");
    }, 1000);
  });
}

// async/await类型安全
async function processData(): Promise<number> {
  try {
    const data = await fetchData(); // 类型为string
    return data.length; // 返回number类型
  } catch (error) {
    console.error("Error processing data:", error);
    return 0;
  }
}

// 复杂异步操作的类型安全
interface User {
  id: number;
  name: string;
  email: string;
}

interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

async function fetchUser(userId: number): Promise<User> {
  const response = await fetch(`/api/users/${userId}`);
  const result: ApiResponse<User> = await response.json();
  
  if (result.status !== 200) {
    throw new Error(result.message);
  }
  
  return result.data;
}

// 并行异步操作
async function fetchMultipleUsers(userIds: number[]): Promise<User[]> {
  // Promise.all保持类型安全
  const users = await Promise.all(
    userIds.map(id => fetchUser(id))
  );
  return users;
}

// 异步生成器
async function* fetchUsersBatch(userIds: number[]): AsyncGenerator<User, void, unknown> {
  for (const id of userIds) {
    yield await fetchUser(id);
  }
}

// 使用异步生成器
async function processUsers() {
  const userIds = [1, 2, 3, 4, 5];
  for await (const user of fetchUsersBatch(userIds)) {
    console.log(user.name);
  }
}
```

#### 2. 什么是映射类型？请举例说明其应用场景

**答案解析**：
映射类型是TypeScript中一种强大的类型操作工具，它通过在现有类型的基础上创建新类型。映射类型使用in关键字遍历现有类型的键来创建新类型。

常见应用场景：
1. **创建只读版本**：将所有属性变为只读
2. **创建可选版本**：将所有属性变为可选
3. **类型转换**：改变属性的类型
4. **键名重映射**：改变键名

```
// 基本映射类型
type OptionsFlags<Type> = {
  [Property in keyof Type]: boolean;
};

type Features = {
  darkMode: () => void;
  newUserProfile: () => void;
};

type FeatureOptions = OptionsFlags<Features>;
// 结果: { darkMode: boolean; newUserProfile: boolean; }

// 使用内置映射类型
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}

// Partial - 将所有属性变为可选
type PartialTodo = Partial<Todo>;
// 等价于: { title?: string; description?: string; completed?: boolean; }

// Required - 将所有属性变为必需
type RequiredTodo = Required<PartialTodo>;
// 等价于原始Todo类型

// Readonly - 将所有属性变为只读
type ReadonlyTodo = Readonly<Todo>;
// const todo: ReadonlyTodo = { title: "Read", description: "Book", completed: false };
// todo.title = "Write"; // 错误：无法分配到"title"，因为它是只读属性

// Record - 创建指定类型键和值的对象类型
type CatInfo = {
  age: number;
  breed: string;
};

type CatName = "miffy" | "boris" | "mordred";

const cats: Record<CatName, CatInfo> = {
  miffy: { age: 10, breed: "Persian" },
  boris: { age: 5, breed: "Maine Coon" },
  mordred: { age: 16, breed: "British Shorthair" },
};

// 自定义映射类型 - 错误处理包装器
type APIResponse<T> = {
  [K in keyof T]: T[K] extends string ? T[K] | null : T[K];
};

interface User {
  id: number;
  name: string;
  email: string;
}

type UserWithOptionalStrings = APIResponse<User>;
// 结果: { id: number; name: string | null; email: string | null; }

// 键名重映射
type Getters<Type> = {
  [Property in keyof Type as `get${Capitalize<string & Property>}`]: () => Type[Property]
};

interface Person {
  name: string;
  age: number;
  location: string;
}

type LazyPerson = Getters<Person>;
// 结果: {
//   getName: () => string;
//   getAge: () => number;
//   getLocation: () => string;
// }

// 条件映射类型
type RemoveKindField<Type> = {
  [Property in keyof Type as Exclude<Property, "kind">]: Type[Property]
};

interface Circle {
  kind: "circle";
  radius: number;
}

interface Square {
  kind: "square";
  sideLength: number;
}

type RemoveKindCircle = RemoveKindField<Circle>;
// 结果: { radius: number; }
```

#### 3. TypeScript中的工具类型有哪些？请举例说明其用途

**答案解析**：
TypeScript提供了许多内置的工具类型，用于常见的类型转换操作。这些工具类型大大简化了类型操作，提高了代码的可维护性。

常用的工具类型包括：

```
// 1. Partial<T> - 将T中的所有属性变为可选
interface User {
  id: number;
  name: string;
  email: string;
}

function updateUser(id: number, updates: Partial<User>): void {
  // 可以只传入需要更新的属性
  console.log(`Updating user ${id} with:`, updates);
}

updateUser(1, { name: "New Name" }); // 只更新name
updateUser(2, { email: "new@example.com", name: "Another Name" }); // 更新多个属性

// 2. Required<T> - 将T中的所有属性变为必需
interface PartialUser {
  id?: number;
  name?: string;
  email?: string;
}

type FullUser = Required<PartialUser>;
// 等价于: { id: number; name: string; email: string; }

// 3. Readonly<T> - 将T中的所有属性变为只读
type ReadonlyUser = Readonly<User>;
const user: ReadonlyUser = { id: 1, name: "John", email: "john@example.com" };
// user.name = "Jane"; // 错误：无法分配到"name"，因为它是只读属性

// 4. Record<K, T> - 创建一个由K类型键和T类型值组成的对象类型
type PageInfo = {
  title: string;
};

type Page = "home" | "about" | "contact";

const nav: Record<Page, PageInfo> = {
  home: { title: "Home" },
  about: { title: "About" },
  contact: { title: "Contact" }
};

// 5. Pick<T, K> - 从T中选择一组属性K
type UserPreview = Pick<User, "id" | "name">;
// 等价于: { id: number; name: string; }

// 6. Omit<T, K> - 从T中排除一组属性K
type UserPublicInfo = Omit<User, "id">;
// 等价于: { name: string; email: string; }

// 7. Exclude<T, U> - 从T中排除可分配给U的类型
type T0 = Exclude<"a" | "b" | "c", "a">; // "b" | "c"
type T1 = Exclude<"a" | "b" | "c", "a" | "b">; // "c"
type T2 = Exclude<string | number | (() => void), Function>; // string | number

// 8. Extract<T, U> - 从T中提取可分配给U的类型
type T3 = Extract<"a" | "b" | "c", "a" | "f">; // "a"
type T4 = Extract<string | number | (() => void), Function>; // () => void

// 9. NonNullable<T> - 从T中排除null和undefined
type T5 = NonNullable<string | number | undefined>; // string | number
type T6 = NonNullable<string[] | null | undefined>; // string[]

// 10. ReturnType<T> - 获取函数类型的返回类型
type T7 = ReturnType<() => string>; // string
type T8 = ReturnType<(s: string) => void>; // void
type T9 = ReturnType<<T>() => T>; // unknown
type T10 = ReturnType<<T extends U, U extends number[]>() => T>; // number[]

// 11. InstanceType<T> - 获取构造函数类型的实例类型
class C {
  x = 0;
  y = 0;
}

type T11 = InstanceType<typeof C>; // C
type T12 = InstanceType<any>; // any
type T13 = InstanceType<never>; // never
// type T14 = InstanceType<string>; // 错误

// 12. Parameters<T> - 获取函数类型的参数类型
type T15 = Parameters<() => string>; // []
type T16 = Parameters<(s: string) => void>; // [string]
type T17 = Parameters<<T>(arg: T) => T>; // [unknown]

// 实际应用示例：创建一个通用的API响应处理函数
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

// 使用工具类型创建不同场景的响应类型
type SuccessResponse<T> = Pick<ApiResponse<T>, "data" | "status">;
type ErrorResponse = Pick<ApiResponse<never>, "status" | "message">;

function handleApiResponse<T>(response: ApiResponse<T>): SuccessResponse<T> | ErrorResponse {
  if (response.status >= 200 && response.status < 300) {
    return {
      data: response.data,
      status: response.status
    };
  } else {
    return {
      status: response.status,
      message: response.message
    };
  }
}
```

#### 4. 如何在React项目中使用TypeScript提高开发效率？

**答案解析**：
在React项目中使用TypeScript可以显著提高开发效率和代码质量，主要体现在以下几个方面：

1. **组件Props类型安全**：确保组件接收正确的props
2. **Hooks类型安全**：useState、useEffect等Hooks的类型推断
3. **事件处理类型安全**：DOM事件和自定义事件的类型检查
4. **Context类型安全**：确保Context值的类型正确

```typescript
// 1. 函数组件Props类型定义
import React, { ReactNode } from 'react';

interface ButtonProps {
  children: ReactNode;
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'small' | 'medium' | 'large';
  disabled?: boolean;
  onClick: () => void;
}

const Button: React.FC<ButtonProps> = ({ 
  children, 
  variant = 'primary', 
  size = 'medium', 
  disabled = false, 
  onClick 
}) => {
  return (
    <button 
      className={`btn btn-${variant} btn-${size} ${disabled ? 'disabled' : ''}`}
      onClick={onClick}
      disabled={disabled}
    >
      {children}
    </button>
  );
};

// 2. 自定义Hook类型安全
import { useState, useEffect } from 'react';

interface UseApiResult<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
  refetch: () => void;
}

function useApi<T>(url: string): UseApiResult<T> {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<Error | null>(null);

  const fetchData = async () => {
    try {
      setLoading(true);
      const response = await fetch(url);
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      const result: T = await response.json();
      setData(result);
    } catch (err) {
      setError(err instanceof Error ? err : new Error('Unknown error'));
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchData();
  }, [url]);

  return { data, loading, error, refetch: fetchData };
}

// 3. 表单处理类型安全
interface UserFormValues {
  name: string;
  email: string;
  age: number;
}

const UserForm: React.FC = () => {
  const [values, setValues] = useState<UserFormValues>({
    name: '',
    email: '',
    age: 0
  });

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setValues(prev => ({
      ...prev,
      [name]: name === 'age' ? Number(value) : value
    }));
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    console.log(values);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input 
        name="name" 
        value={values.name} 
        onChange={handleChange} 
        placeholder="Name" 
      />
      <input 
        name="email" 
        value={values.email} 
        onChange={handleChange} 
        placeholder="Email" 
        type="email"
      />
      <input 
        name="age" 
        value={values.age} 
        onChange={handleChange} 
        placeholder="Age" 
        type="number"
      />
      <button type="submit">Submit</button>
    </form>
  );
};

// 4. Context类型安全
import { createContext, useContext } from 'react';

interface ThemeContextType {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export const ThemeProvider: React.FC<{ children: ReactNode }> = ({ children }) => {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');
  
  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

export const useTheme = (): ThemeContextType => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
};
```

### 9.4 TypeScript 5.5新特性面试题

#### 1. TypeScript 5.5中推断类型谓词是如何改进的？

**答案解析**：
TypeScript 5.5改进了类型谓词的推断能力，特别是在数组过滤操作中能够自动推断更精确的类型。在之前的版本中，即使通过过滤器明确筛选了特定类型的元素，TypeScript仍然会保持原来的联合类型，需要手动进行类型转换。

```typescript
// TypeScript 5.5之前的版本
interface Bird {
  commonName: string;
  scientificName: string;
  sing(): void;
}

declare const nationalBirds: Map<string, Bird>;

function makeBirdCalls(countries: string[]) {
  // 在TypeScript 5.5之前：birds类型为(Bird | undefined)[]
  const birds = countries
    .map(country => nationalBirds.get(country))
    .filter(bird => bird !== undefined);

  // 需要类型断言才能调用sing方法
  for (const bird of birds) {
    (bird as Bird).sing(); // 需要类型断言
  }
}

// 数组过滤的例子
const namesAndAges = ["Elijah", "Sophia", "Liam", "Isabella", "Mason", 23, 24];
const ages = namesAndAges.filter(age => typeof age === 'number');
// TypeScript 5.5之前：ages的类型为(string | number)[]
// ages.forEach((age) => console.log(age + 1)); // 错误：不能保证age是number类型

// TypeScript 5.5及以后版本
function makeBirdCallsNew(countries: string[]) {
  // 在TypeScript 5.5之后：birds类型自动推断为Bird[]
  const birds = countries
    .map(country => nationalBirds.get(country))
    .filter(bird => bird !== undefined);

  // 不再需要类型断言
  for (const bird of birds) {
    bird.sing(); // 直接可以调用，类型安全
  }
}

// 数组过滤的例子 - 改进后
const namesAndAgesNew = ["Elijah", "Sophia", "Liam", "Isabella", "Mason", 23, 24];
const agesNew = namesAndAgesNew.filter(age => typeof age === 'number');
// TypeScript 5.5之后：agesNew的类型自动推断为number[]
agesNew.forEach((age) => console.log(age + 1)); // 直接可以使用，类型安全
```

#### 2. TypeScript 5.5如何改进常量索引访问的控制流收窄？

**答案解析**：
TypeScript 5.5增强了对形式为obj[key]的表达式的类型收窄能力，当obj和key都是有效的常量时。在之前的版本中，即使通过typeof检查了属性的类型，TypeScript仍然不能确定该属性的具体类型，需要使用中间变量来辅助类型推断。

```typescript
// TypeScript 5.5之前的版本
function downCaseOld(obj: Record<string, number | string>, key: string) {
  // 之前的版本不能正确收窄类型
  if (typeof obj[key] === "string") {
    // obj[key].toLowerCase(); // 错误：TypeScript不能确定obj[key]是string类型
    // 需要使用中间变量
    const value = obj[key];
    if (typeof value === "string") {
      console.log(value.toLowerCase()); // 正确
    }
  }
}

// TypeScript 5.5及以后版本
function downCaseNew(obj: Record<string, number | string>, key: string) {
  // 现在可以直接使用，TypeScript能正确收窄类型
  if (typeof obj[key] === "string") {
    console.log(obj[key].toLowerCase()); // 正确，无需中间变量
  }
}

// 更多例子
interface ApiResponse {
  data: string | number;
  status: number;
  message?: string;
}

function processResponse(response: ApiResponse, key: 'data' | 'message') {
  // TypeScript 5.5可以更好地处理这种情况
  if (key === 'data' && typeof response[key] === 'string') {
    console.log(response[key].toUpperCase()); // 正确，类型已收窄为string
  }
  
  if (key === 'message' && response[key]) {
    console.log(response[key].length); // 正确，类型已收窄为string
  }
}
```

#### 3. TypeScript 5.5对正则表达式语法检查有哪些改进？

**答案解析**：
TypeScript 5.5增加了对正则表达式字面量的基本语法检查功能。在之前的版本中，TypeScript不会检查正则表达式的语法正确性，这可能导致运行时错误。新版本会在编译时检测一些常见的正则表达式语法错误。

```typescript
// TypeScript 5.5之前的版本
// 以下错误的正则表达式不会被检测到
let myRegex1 = /@robot(\s+(please|immediately)))? do some task/; // 多余的括号
let myRegex2 = /@typedef \{import\((.+)\)\.([a-zA-Z_]+)\} \3/u; // 无效的反向引用

// TypeScript 5.5及以后版本
// 以下错误会被编译器检测到并报错
// let myRegex1 = /@robot(\s+(please|immediately)))? do some task/;
//                                            ~ 错误：意外的')'，你是否想用反斜杠转义？

// let myRegex2 = /@typedef \{import\((.+)\)\.([a-zA-Z_]+)\} \3/u;
//                                                        ~ 错误：这个反向引用指向一个不存在的组

// 正确的正则表达式
let validRegex1 = /@robot(\s+(please|immediately))? do some task/;
let validRegex2 = /@typedef \{import\((.+)\)\.([a-zA-Z_]+)\} \2/u;

// 注意：TypeScript的正则表达式检查仅限于正则表达式字面量
// 使用RegExp构造函数时不会进行语法检查
let regexFromString = new RegExp("@robot(\\s+(please|immediately)))? do some task"); // 不会报错
```

#### 4. TypeScript 5.5对ECMAScript Set新方法的支持

**答案解析**：
TypeScript 5.5增加了对ECMAScript新标准中Set对象新方法的支持，包括union、intersection、difference和symmetricDifference等方法。这些方法使得集合操作更加直观和强大。

```typescript
// 需要在tsconfig.json中配置lib选项以支持这些新方法
// {
//   "compilerOptions": {
//     "lib": ["ES2025", "DOM"]
//   }
// }

// Set的新方法示例
let primaryColors = new Set(["red", "blue", "yellow"]);
let secondaryColors = new Set(["green", "orange", "purple"]);
let warmColors = new Set(["red", "orange", "yellow"]);

// 并集 - union方法
let allColors = primaryColors.union(secondaryColors);
console.log(allColors); // Set { 'red', 'blue', 'yellow', 'green', 'orange', 'purple' }

// 交集 - intersection方法
let warmPrimaryColors = primaryColors.intersection(warmColors);
console.log(warmPrimaryColors); // Set { 'red', 'yellow' }

// 差集 - difference方法
let coolPrimaryColors = primaryColors.difference(warmColors);
console.log(coolPrimaryColors); // Set { 'blue' }

// 对称差集 - symmetricDifference方法
let nonWarmColors = primaryColors.symmetricDifference(warmColors);
console.log(nonWarmColors); // Set { 'blue' }

// 实际应用示例：权限管理系统
class PermissionManager {
  private userPermissions: Set<string>;
  private rolePermissions: Map<string, Set<string>>;
  
  constructor() {
    this.userPermissions = new Set();
    this.rolePermissions = new Map();
  }
  
  addRolePermissions(role: string, permissions: string[]) {
    this.rolePermissions.set(role, new Set(permissions));
  }
  
  assignRoleToUser(role: string) {
    const rolePermissions = this.rolePermissions.get(role);
    if (rolePermissions) {
      // 使用union方法合并权限
      this.userPermissions = this.userPermissions.union(rolePermissions);
    }
  }
  
  revokeRoleFromUser(role: string) {
    const rolePermissions = this.rolePermissions.get(role);
    if (rolePermissions) {
      // 使用difference方法移除权限
      this.userPermissions = this.userPermissions.difference(rolePermissions);
    }
  }
  
  hasPermission(permission: string): boolean {
    return this.userPermissions.has(permission);
  }
  
  getPermissions(): Set<string> {
    return new Set(this.userPermissions);
  }
}

// 使用示例
const pm = new PermissionManager();
pm.addRolePermissions('admin', ['read', 'write', 'delete', 'manage_users']);
pm.addRolePermissions('editor', ['read', 'write', 'edit_content']);
pm.addRolePermissions('viewer', ['read']);

pm.assignRoleToUser('editor');
console.log(pm.getPermissions()); // Set { 'read', 'write', 'edit_content' }

pm.assignRoleToUser('admin');
console.log(pm.getPermissions()); // Set { 'read', 'write', 'edit_content', 'delete', 'manage_users' }
```

### 9.5 TypeScript性能优化与最佳实践面试题

#### 1. TypeScript 7.0（原生Go版本）的性能提升有哪些重要意义？

**答案解析**：
TypeScript 7.0是TypeScript发展的一个重要里程碑，Microsoft正在将TypeScript编译器从JavaScript移植到Go语言，预计性能将提升10倍。这对于大型项目和开发体验有着深远的影响。

主要性能提升包括：
1. **编译速度提升**：预计提升10倍，大幅减少等待时间
2. **内存使用优化**：更低的内存占用，更好的资源利用
3. **启动时间缩短**：编辑器和IDE的响应速度显著提升
4. **增量编译优化**：更快的增量构建和热重载

```bash
# 性能对比（早期测试数据）
# Codebase     | Size (LOC) | Current | Native | Speedup
# VS Code      | 1,505,000  | 77.8s   | 7.5s   | 10.4x
# Playwright   | 356,000    | 11.1s   | 1.1s   | 10.1x
# TypeORM      | 270,000    | 17.5s   | 1.3s   | 13.5x

# 实际项目中的性能优化示例
// 1. 合理使用tsconfig.json配置
{
  "compilerOptions": {
    // 启用增量编译
    "incremental": true,
    "tsBuildInfoFile": "./dist/tsconfig.tsbuildinfo",
    
    // 项目引用（Project References）
    "composite": true,
    
    // 跳过不必要的检查
    "skipLibCheck": true,
    "skipDefaultLibCheck": true,
    
    // 优化模块解析
    "moduleResolution": "node",
    "resolveJsonModule": true,
    
    // 严格模式按需启用
    "strict": true,
    // "noImplicitAny": true,
    // "strictNullChecks": true,
    
    // 输出优化
    "noEmitOnError": false,
    "removeComments": true
  },
  
  // 使用references进行项目引用
  "references": [
    { "path": "./shared" },
    { "path": "./modules/user" },
    { "path": "./modules/product" }
  ]
}

// 2. 避免过度使用复杂的条件类型
// 不推荐：过于复杂的条件类型会影响编译性能
type ComplexType<T> = T extends infer U 
  ? U extends object 
    ? U extends any[] 
      ? U[number] extends infer V 
        ? V extends object 
          ? ComplexType<V> 
          : V
        : never
      : { [K in keyof U]: ComplexType<U[K]> }
    : T
  : never;

// 推荐：简化类型逻辑
type SimplifiedType<T> = T extends (infer U)[]
  ? U
  : T extends object
    ? { [K in keyof T]: SimplifiedType<T[K]> }
    : T;
```

#### 2. 在大型项目中如何优化TypeScript的编译性能？

**答案解析**：
在大型项目中，TypeScript的编译性能优化至关重要。以下是一些有效的优化策略：

```typescript
// 1. 项目结构优化
// 合理划分模块，避免循环依赖
// 使用monorepo结构管理大型项目
// 示例：合理的项目结构
/*
src/
├── shared/           # 共享类型和工具
│   ├── types/
│   └── utils/
├── modules/          # 功能模块
│   ├── user/
│   ├── product/
│   └── order/
├── services/         # 服务层
└── components/       # 组件库
*/

// 2. tsconfig.json优化配置
{
  "compilerOptions": {
    // 启用增量编译
    "incremental": true,
    "tsBuildInfoFile": "./dist/tsconfig.tsbuildinfo",
    
    // 项目引用（Project References）
    "composite": true,
    
    // 跳过不必要的检查
    "skipLibCheck": true,
    "skipDefaultLibCheck": true,
    
    // 优化模块解析
    "moduleResolution": "node",
    "resolveJsonModule": true,
    
    // 严格模式按需启用
    "strict": true,
    // "noImplicitAny": true,
    // "strictNullChecks": true,
    
    // 输出优化
    "noEmitOnError": false,
    "removeComments": true
  },
  
  // 使用references进行项目引用
  "references": [
    { "path": "./shared" },
    { "path": "./modules/user" },
    { "path": "./modules/product" }
  ]
}

// 3. 类型优化：避免过度复杂的泛型
// 不推荐：嵌套过深的泛型
type DeepNested<T> = {
  [K in keyof T]: T[K] extends object
    ? DeepNested<T[K]> extends infer U
      ? U extends object
        ? U & { _id: string }
        : U
      : never
    : T[K];
};

// 推荐：扁平化处理
type FlattenObject<T> = {
  [K in keyof T]: T[K] extends object ? T[K] : T[K];
};

// 4. 合理使用索引签名
// 不推荐：过于宽泛的索引签名
interface BadExample {
  [key: string]: any; // 失去类型安全
}

// 推荐：明确的索引签名
interface GoodExample {
  [key: string]: string | number | boolean;
  // 或者使用Record工具类型
}

type BetterExample = Record<string, string | number | boolean>;

// 5. 避免在类型中使用运行时值
const API_ENDPOINTS = {
  users: '/api/users',
  products: '/api/products'
} as const;

// 不推荐：在类型中使用运行时值
type Endpoints = typeof API_ENDPOINTS[keyof typeof API_ENDPOINTS];

// 推荐：明确定义类型
type Endpoint = '/api/users' | '/api/products';
```

#### 3. TypeScript中的类型安全最佳实践有哪些？

**答案解析**：
类型安全是TypeScript的核心价值，以下是一些关键的最佳实践：

```typescript
// 1. 避免使用any，优先使用unknown
// 不好的做法
function processValueBad(value: any) {
  return value.toFixed(2); // 运行时可能出错
}

// 好的做法
function processValueGood(value: unknown) {
  if (typeof value === 'number') {
    return value.toFixed(2);
  }
  throw new Error('Value is not a number');
}

// 2. 使用satisfies操作符确保类型兼容性
type Theme = 'light' | 'dark';
const themeConfig = {
  light: { bgColor: '#fff', textColor: '#000' },
  dark: { bgColor: '#000', textColor: '#fff' }
} satisfies Record<Theme, { bgColor: string; textColor: string }>;

// 3. 合理使用类型守卫
function isString(value: unknown): value is string {
  return typeof value === 'string';
}

function isArray<T>(value: unknown): value is T[] {
  return Array.isArray(value);
}

// 4. 使用const断言创建不可变类型
const colors = {
  red: '#ff0000',
  green: '#00ff00',
  blue: '#0000ff'
} as const;

type Color = keyof typeof colors; // "red" | "green" | "blue"
type ColorValue = typeof colors[keyof typeof colors]; // "#ff0000" | "#00ff00" | "#0000ff"

// 5. 使用模板字面量类型创建精确的字符串类型
type HttpMethod = 'GET' | 'POST' | 'PUT' | 'DELETE';
type ApiEndpoint = `/api/${string}`;

function createApiUrl(method: HttpMethod, endpoint: ApiEndpoint): string {
  return `${method} ${endpoint}`;
}

// 6. 合理使用工具类型
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}

// 使用Omit排除敏感信息
type PublicUser = Omit<User, 'password'>;

// 使用Pick选择需要的属性
type UserPreview = Pick<User, 'id' | 'name'>;

// 使用Partial创建可选版本
type UpdateUserDto = Partial<User>;

// 7. 创建自定义工具类型
// 确保对象具有必需的属性
type RequireAtLeastOne<T, Keys extends keyof T = keyof T> = 
  Pick<T, Exclude<keyof T, Keys>> & 
  {
    [K in Keys]-?: Required<Pick<T, K>> & Partial<Pick<T, Exclude<Keys, K>>>
  }[Keys];

interface UserContact {
  email?: string;
  phone?: string;
  address?: string;
}

// 至少需要提供一种联系方式
type UserWithContact = RequireAtLeastOne<UserContact, 'email' | 'phone' | 'address'>;

const user1: UserWithContact = { email: 'user@example.com' }; // 正确
const user2: UserWithContact = { phone: '123-456-7890' }; // 正确
// const user3: UserWithContact = {}; // 错误：至少需要一个联系方式