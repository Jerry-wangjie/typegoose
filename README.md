# Typegoose

[![Build Status](https://travis-ci.org/szokodiakos/typegoose.svg?branch=master)](https://travis-ci.org/szokodiakos/typegoose)
[![Coverage Status](https://coveralls.io/repos/github/szokodiakos/typegoose/badge.svg?branch=master)](https://coveralls.io/github/szokodiakos/typegoose?branch=master)
[![npm](https://img.shields.io/npm/dt/typegoose.svg)]()

Define Mongoose models using TypeScript classes.

使用TypeScript类定义Mongoose模型。

## Basic usage

```typescript
import { prop, Typegoose, ModelType, InstanceType } from 'typegoose';
import * as mongoose from 'mongoose';

mongoose.connect('mongodb://localhost:27017/test');

class User extends Typegoose {
  @prop()
  name?: string;
}

const UserModel = new User().getModelForClass(User);

// UserModel is a regular Mongoose Model with correct types
(async () => {
  const u = new UserModel({ name: 'JohnDoe' });
  await u.save();
  const user = await UserModel.findOne();

  // prints { _id: 59218f686409d670a97e53e0, name: 'JohnDoe', __v: 0 }
  console.log(user);
})();
```

## Motivation

A common problem when using Mongoose with TypeScript is that you have to define both the Mongoose model and the TypeScript interface. If the model changes, you also have to keep the TypeScript interface file in sync or the TypeScript interface would not represent the real data structure of the model.

在TypeScript中使用Mongoose时，常见的问题是您必须同时定义Mongoose模型和TypeScript接口。 如果模型更改，则还必须保持TypeScript接口文件同步，否则TypeScript接口将不会表示模型的实际数据结构。

Typegoose aims to solve this problem by defining only a TypeScript interface (class) which need to be enhanced with special Typegoose decorators.

Typegoose旨在通过仅定义一个TypeScript接口（类）来解决这个问题，这个接口需要用特殊的TypeGo装饰器来增强。

Under the hood it uses the [reflect-metadata](https://github.com/rbuckton/reflect-metadata) API to retrieve the types of the properties, so redundancy can be significantly reduced.

在底层，它使用[reflect-metadata]API来检索属性的类型，因此可以显着减少冗余。

Instead of:
替换:
```typescript
interface Car {
  model?: string;
}

interface Job {
  title?: string;
  position?: string;
}

interface User {
  name?: string;
  age: number;
  job?: Job;
  car: Car | string;
}

mongoose.model('User', {
  name: String,
  age: { type: Number, required: true },
  job: {
    title: String;
    position: String;
  },
  car: { type: Schema.Types.ObjectId, ref: 'Car' }
});

mongoose.model('Car', {
  model: string,
});
```
You can just:
你只需要这样:
```typescript
class Job {
  @prop()
  title?: string;

  @prop()
  position?: string;
}

class Car extends Typegoose {
  @prop()
  model?: string;
}

class User extends Typegoose {
  @prop()
  name?: string;

  @prop({ required: true })
  age: number;

  @prop()
  job?: Job;

  @prop({ ref: Car, required: true })
  car: Ref<Car>;
}
```
Please note that sub documents doesn't have to extend Typegoose. You can still give them default value in `prop` decorator, but you can't create static or instance methods on them.

请注意，子文件不必继承Typegoose。 你仍然可以在`prop`装饰器中给它们默认值，但是你不能在它们上面创建静态或实例方法。

## Requirements(要求)

* TypeScript 2.1+
* `emitDecoratorMetadata` and `experimentalDecorators` must be enabled in `tsconfig.json`

## Install(安装)

`npm install typegoose -S`

## Testing(测试)

`npm test`

## API Documentation(API说明)

### Typegoose class(类)

This is the class which your schema defining classes must extend.

这是您的模式定义类必须扩展的类。

#### Methods:(方法)

`getModelForClass<T>(t: T, options?: GetModelForClassOptions)`

This method returns the corresponding Mongoose Model for the class (`T`). If no Mongoose model exists for this class yet, one will be created automatically (by calling the method `setModelForClass`).

这个方法返回类（`T`）对应的Mongoose模型。 如果这个类还没有Mongoose模型，将会自动创建一个模型（通过调用setModelForClass方法）。


`setModelForClass<T>(t: T, options?: GetModelForClassOptions)`

This method assembles the Mongoose Schema from the decorated schema defining class, creates the Mongoose Model and returns it. For typing reasons, the schema defining class must be passed down to it.

该方法从装饰模式定义类中组装Mongoose模式，创建Mongoose模型并返回它。 为了输入的原因，模式定义类必须传递给它。

Hint: If a Mongoose Model already exists for this class, it will be overwritten.

提示：如果这个类已经存在一个Mongoose模型，它将被覆盖。


The `GetModelForClassOptions` provides multiple optional configurations:(提供了多个可选配置)
 * `existingMongoose: mongoose`: An existing Mongoose instance can also be passed down. If given, Typegoose uses this Mongoose instance's `model` methods.(现有的Mongoose实例也可以传递下去。 如果给定，Typegoose使用这个Mongoose实例的`model`方法。)
 
 * `schemaOptions: mongoose.SchemaOptions`: Additional [schema options](http://mongoosejs.com/docs/guide.html#options) can be passed down to the schema-to-be-created.(可以将其他传递给要创建的模式。)
 
 * `existingConnection: mongoose.Connection`: An existing Mongoose connection can also be passed down. If given, Typegoose uses this Mongoose instance's `model` methods.(现有的Mongoose连接也可以传递下去。 如果给定，Typegoose使用这个Mongoose实例的`model`方法。)

### Property decorators(属性装饰)

Typegoose comes with TypeScript decorators, which responsibility is to connect the Mongoose schema behind the TypeScript class.

Typegoose带有TypeScript装饰器，它负责连接TypeScript类后面的Mongoose模式。

#### prop(options)

The `prop` decorator adds the target class property to the Mongoose schema as a property. Typegoose checks the decorated property's type and sets the schema property accordingly. If another Typegoose extending class is given as the type, Typegoose will recognize this property as a sub document.

“prop”装饰器将目标类属性作为属性添加到Mongoose模式。 Typegoose检查装饰属性的类型并相应地设置模式属性。 如果给出另一个Typegoose扩展类作为类型，则Typegoose将把该属性识别为子文档。

The `options` object accepts multiple config properties:(`options`对象接受多个配置属性)
  - `required`: Just like the [Mongoose required](http://mongoosejs.com/docs/api.html#schematype_SchemaType-required)
    it accepts a handful of parameters. Please note that it's the developer's responsibility to make sure that
    
    (就像[Mongoose required]一样,它接受一些参数。 请注意，开发者有责任确保这一点)
    
    if `required` is set to `false` then the class property should be `optional`
    
    如果`required`设置为`false`，那么class属性应该是`optional`

```typescript
// this is now required in the schema
@prop({ required: true })
firstName: string;

// by default, a property is not required
@prop()
lastName?: string; // using the ? optional property
```

  - `index`: Tells Mongoose whether to define an index for the property.
  告诉Mongoose是否为属性添加索引

```typescript
@prop({ index: true })
indexedField?: string;
```

  - `unique`: Just like the [Mongoose unique](http://mongoosejs.com/docs/api.html#schematype_SchemaType-unique), tells Mongoose to ensure a unique index is created for this path.(唯一索引)

```typescript
// this field is now unique across the collection
@prop({ unique: true })
uniqueId?: string;
```

```typescript
// this field is now unique across the collection
@prop({ unique: true })
uniqueId?: string;
```

  - `enum`: The enum option accepts a string array. The class property which gets this decorator should have an enum-like type which values are from the provided string array. The way how the enum is created is delegated to the developer, Typegoose needs a string array which hold the enum values, and a TypeScript type which tells the possible values of the enum.
  However, if you use TS 2.4+, you can use string enum as well.
  
  枚举选项接受一个字符串数组。 得到这个装饰器的类属性应该有一个枚举类型，它的值来自提供的字符串数组。 如何创建枚举的方式委托给开发人员，Typegoose需要一个包含枚举值的字符串数组，以及一个TypeScript类型，它告诉枚举的可能值。
   但是，如果您使用TS 2.4+，则也可以使用字符串枚举。

```typescript
// Enum-like type and definition example.
type Gender = 'male' | 'female';
const Genders = {
  MALE: 'male' as Gender,
  FEMALE: 'female' as Gender,
};

@prop({ enum: Object.values(Genders) })
gender?: Gender;


// TS 2.4+ string enum example
enum Gender {
  MALE = 'male',
  FEMALE = 'female',
}

@prop({ enum: Gender })
gender?: Gender;
```

  - `default`: The provided value will be the default for that Mongoose property.提供的值将是该Mongoose属性的默认值。

```typescript
@prop({ default: 'Nick' })
nickName?: string;
```

  - `ref`: By adding the `ref` option with another Typegoose class as value, a Mongoose reference property will be created. The type of the property on the Typegoose extending class should be `Ref<T>` (see Types section).
  
  通过将`ref`选项与另一个Typegoose类相加作为值，将创建一个Mongoose引用属性。 Typegoose扩展类的属性类型应该是`Ref <T>`（参见类型部分）。

```typescript
class Car extends Typegoose {}

@prop({ ref: Car })
car?: Ref<Car>;
```

  - `min` / `max` (numeric validators): Same as [Mongoose numberic validators](http://mongoosejs.com/docs/api.html#schema_number_SchemaNumber-max).(数字验证器)

```typescript
@prop({ min: 10, max: 21 })
age?: number;
```

  - `minlength` / `maxlength` / `match` (string validators): Same as [Mongoose string validators](http://mongoosejs.com/docs/api.html#schema_string_SchemaString-match).(字符串验证器)

```typescript
@prop({ minlength: 5, maxlength: 10, match: /[0-9a-f]*/ })
favouriteHexNumber: string;
```

Mongoose gives developers the option to create [virtual properties](http://mongoosejs.com/docs/api.html#schema_Schema-virtual). This means that actual database read/write will not occur these are just 'calculated properties'. A virtual property can have a setter and a getter. TypeScript also has a similar feature which Typegoose uses for virtual property definitions (using the `prop` decorator).

Mongoose为开发人员提供了创建[虚拟属性]的选项。 这意味着实际的数据库读写不会发生，只是“计算属性”。 一个虚拟属性可以有一个setter和一个getter。 TypeScript也有类似的功能，Typegoose使用虚拟属性定义（使用`prop`装饰器）。

```typescript
@prop()
firstName?: string;

@prop()
lastName?: string;

@prop() // this will create a virtual property called 'fullName' 创建了虚拟属性fullName
get fullName() {
  return `${this.firstName} ${this.lastName}`;
}
set fullName(full) {
  const [firstName, lastName] = full.split(' ');
  this.firstName = firstName;
  this.lastName = lastName;
}
```

#### arrayProp(options)

The `arrayProp` is a `prop` decorator which makes it possible to create array schema properties.

`arrayProp`是一个`prop`装饰器，它可以创建数组模式属性。

The `options` object accepts `required`, `enum` and `default`, just like the `prop` decorator. In addition to these the following properties exactly one should be given:

“options”对象接受`required`，`enum`和`default`，就像`prop`装饰器一样。 除了这些，还应该给出以下属性：

  - `items`: This will tell Typegoose that this is an array which consists of primitives (if `String`, `Number`, or other primitive type is given) or this is an array which consists of subdocuments (if it's extending the `Typegoose` class).
  
  这将告诉Typegoose，这是一个由基元构成的数组（如果给出`String`，`Number`或其他基本类型），或者这是一个由子文档构成的数组（如果它扩展了`Typegoose`类）

```typescript
@arrayProp({ items: String })
languages?: string[];
```

Note that unfortunately the [reflect-metadata](https://github.com/rbuckton/reflect-metadata) API does not let us determine the type of the array, it only returns `Array` when the type of the property is queried. This is why redundancy is required here.

请注意，不幸的是，[reflect-metadata]API不让我们确定数组的类型，只有在查询属性的类型时才返回`Array`。 这就是为什么这里需要冗余。

  - `itemsRef`: In mutual exclusion with `items`, this tells Typegoose that instead of a subdocument array, this is an array with references in it. On the Mongoose side this means that an array of Object IDs will be stored under this property. Just like with `ref` in the `prop` decorator, the type of this property should be `Ref<T>[]`.
  
  在与`items`互斥的情况下，这告诉Typegoose，而不是一个子文档数组，这是一个带有引用的数组。 在Mongoose方面，这意味着一个对象ID数组将被存储在这个属性下。 就像`prop`装饰器中的`ref`一样，这个属性的类型应该是`Ref <T> []`。

```typescript
class Car extends Typegoose {}

@arrayProp({ itemsRef: Car })
previousCars?: Ref<Car>[];
```

### Method decorators(方法装饰器)

In Mongoose we can attach two types of methods for our schemas: static (model) methods and instance methods. Both of them are supported by Typegoose.

在Mongoose中，我们可以为我们的模式附加两种类型的方法：静态（模型）方法和实例方法。 两者都是由Typegoose支持的。

#### staticMethod(静态方法)

Static Mongoose methods must be declared with `static` keyword on the Typegoose extending class. This will ensure, that these methods are callable on the Mongoose model (TypeScript won't throw development-time error for unexisting method on model object).

静态Mongoose方法必须在Typegoose扩展类中用`static`关键字声明。 这将确保这些方法在Mongoose模型上可调用（TypeScript不会为模型对象上的未知方法抛出开发时错误）。

If we want to use another static method of the model (built-in or created by us) we have to override the `this` in the method using the [type specifying of `this` for functions](https://github.com/Microsoft/TypeScript/wiki/What%27s-new-in-TypeScript#specifying-the-type-of-this-for-functions). If we don't do this, TypeScript will throw development-time error on missing methods.

如果我们想要使用模型的另一个静态方法（内置或由我们创建），我们必须使用指定`this`的类型来覆盖函数中的`this`。 如果我们不这样做，TypeScript将在缺少的方法上抛出开发时错误。

```typescript
@staticMethod
static findByAge(this: ModelType<User> & typeof User, age: number) {
  return this.findOne({ age });
}
```

Note that the `& typeof T` is only mandatory if we want to use the developer defined static methods inside this static method. If not then the `ModelType<T>` is sufficient, which will be explained in the Types section.

请注意，如果我们想在这个静态方法中使用开发者定义的静态方法，`＆typeof T`只是必须的。 如果不是那么`ModelType <T>`就足够了，这将在类型部分中解释。

#### instanceMethod(实例方法)

Instance methods are on the Mongoose document instances, thus they must be defined as non-static methods. Again if we want to call other instance methods the type of `this` must be redefined to `InstanceType<T>` (see Types).

实例方法在Mongoose文档实例上，因此它们必须被定义为非静态方法。 同样，如果我们想调用其他实例方法，则必须将`this`的类型重新定义为`InstanceType <T>`（参见类型）。

```typescript
@instanceMethod
incrementAge(this: InstanceType<User>) {
  const age = this.age || 1;
  this.age = age + 1;
  return this.save();
}
```

### Class decorators(类装饰器)

Mongoose allows the developer to add pre and post [hooks / middlewares](http://mongoosejs.com/docs/middleware.html) to the schema. With this it is possible to add document transformations and observations before or after validation, save and more.

Mongoose允许开发人员在模式中添加前后中间件。 有了这个，可以在验证之前或之后添加文档转换和观察，保存等等。

Typegoose provides this functionality through TypeScript's class decorators.

Typegoose通过TypeScript的类装饰器提供了这个功能。

### pre(前置)

We can simply attach a `@pre` decorator to the Typegoose class and define the hook function like you normally would in Mongoose.

我们可以简单地将`@ pre`装饰器附加到Typegoose类中，并像在Mongoose中一样定义钩子函数。

```typescript
@pre<Car>('save', function(next) { // or @pre(this: Car, 'save', ...
  if (this.model === 'Tesla') {
    this.isFast = true;
  }
  next();
})
class Car extends Typegoose {
  @prop({ required: true })
  model: string;

  @prop()
  isFast: boolean;
}
```

This will execute the pre-save hook each time a `Car` document is saved. Inside the pre-hook Mongoose binds the actual document to `this`.

每次保存“Car”文件时，这将执行预保存钩子。 在pre-hook里面，Mongoose将实际的文档绑定到`this`。

Note that additional typing information is required either by passing the class itself as a type parameter `<Car>` or explicity telling TypeScript that `this` is a `Car` (`this: Car`). This will grant typing informations inside the hook function.

请注意，通过将类本身作为类型参数“<Car>”或“显式地告诉TypeScript”，“this”是“Car”（`this：Car`），可以使用附加的类型信息。 这将允许在钩子函数内部输入信息。

#### post(后置)

Same as `pre`, the `post` hook is also implemented as a class decorator. Usage is equivalent with the one Mongoose provides.

和`pre`一样，`post`钩子也是作为类装饰器实现的。 用法与Mongoose提供的相同。

```typescript
@post<Car>('save', (car) => { // or @post('save', (car: Car) => { ...
  if (car.topSpeedInKmH > 300) {
    console.log(car.model, 'is fast!');
  }
})
class Car extends Typegoose {
  @prop({ required: true })
  model: string;

  @prop({ required: true })
  topSpeedInKmH: number;
}
```

Of course `this` is not the document in a post hook (see Mongoose docs). Again typing information is required either by explicit parameter typing or by providing a template type.

当然`this`不是文档后置钩子（请参阅Mongoose文档）。 通过明确的参数输入或提供一个模板类型，再次输入信息是必需的。

#### plugin(插件)

Using the `plugin` decorator enables the developer to attach various Mongoose plugins to the schema. Just like the regular `schema.plugin()` call, the decorator accepts 1 or 2 parameters: the plugin itself, and an optional configuration object. Multiple `plugin` decorator can be used for a single Typegoose class.

使用`plugin`装饰器，开发人员可以将各种Mongoose插件附加到架构中。 就像常规的`schema.plugin（）`调用一样，装饰器接受1或2个参数：插件本身和一个可选的配置对象。 多个`plugin`装饰器可以用于单个Typegoose类。

If the plugin enhances the schema with additional properties or instance / static methods this typing information should be added manually to the Typegoose class as well.

如果插件使用其他属性或实例/静态方法增强架构，则此类型信息也应手动添加到Typegoose类中。

```typescript
import * as findOrCreate from 'mongoose-findorcreate';

@plugin(findOrCreate)
class User extends Typegoose {
  // this isn't the complete method signature, just an example
  static findOrCreate(condition: InstanceType<User>):
    Promise<{ doc: InstanceType<User>, created: boolean }>;
}

const UserModel = new User().getModelForClass(User);
UserModel.findOrCreate({ ... }).then(findOrCreateResult => {
  ...
});
```

### Types(类型)

Some additional types were added to make Typegoose more user friendly.

增加了一些额外的类型，使Typegoose更加用户友好。

#### InstanceType<T>

This is basically the logical 'and' of the `T` and the `mongoose.Document`, so that both the Mongoose instance properties/functions and the user defined properties/instance methods are available on the instance.

这基本上是“T”和“mongoose.Document”的逻辑“和”，所以Mongoose实例属性/函数和用户定义的属性/实例方法在实例上都可用。

#### ModelType<T>

This is the logical 'and' of `mongoose.Model<InstanceType<T>>` and `T`, so that the Mongoose model creates `InstanceType<T>` typed instances and all user defined static methods are available on the model.

这是`mongoose.Model <InstanceType <T >>`和`T`的逻辑“和”，所以Mongoose模型创建`InstanceType <T>`类型实例，并且所有用户定义的静态方法在模型中都可用。

#### Ref<T>

`Ref<T>` means `T` logical 'or' `string`, so that both populated and unpopulated scenarios are handled for the reference property.

`Ref <T>`表示`T`逻辑"或" `string`，因此对于引用属性处理填充和未填充的场景。

## Improvements(改进)

* Add frequently used (currently not present) features if needed(如果需要添加经常使用（目前不存在）的功能)
* Create moar tests (break down current huge one into multiple unit tests)(创建moar测试（将当前巨大的测试分解成多个单元测试）)
