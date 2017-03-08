#Dagger

[原文链接](https://google.github.io/dagger/)

Dagger是一个完全静态，编译时期的依赖注入框架，支持Java和Android。它是Square创建的早期版本的改编版，现在由Google维护。

Dagger用来处理很多基于反射解决方案的开发和性能问题。

#User's Guide
应用中一个好的类是那些处理某个事务的，像BarcodeDecoder，KoopaPhysicEngine以及AudioStreamer。这些类都有依赖，或许是一个BarcodeCameraFinder，DefaultPhysicsEngine或者HttpStreamer。

与之对比的，应用中一个那些差的类就是占用空间却不做任何事情，例如BarcodeDecoderFactory，CameraServiceLoader以及MutableContextWrapper。这些类非常笨拙地与感兴趣的事务缠在一起。

Dagger是这些Factory类的替换方案，实现了依赖注入设计模式，摆脱了写模板代码的负担。它使得你聚焦感兴趣的类。声明依赖，指定如何满足它们，然后运载你的应用。

通过基于标准的javax.inject注解构建，每个类都易于测试。你不需要一系列的模板来去仅仅置换RpcCreditCardService和FakeCreditCardService。

依赖注入不仅仅是为了测试，它同样使得创建可复用、可互换模型变得容易。你可以在应用中共享同样的AuthenticationModule，你也可以在开发阶段运行DevLoggingModule而在产品上线阶段运行ProdLoggingModule从而在每个情形下获得正确的行为。

##Why Dagger2 is Different
依赖注入框架已经存在很多年了，并且拥有了完整的、各种API来配置和注入。那么，为什么还要重新创造轮子呢？Dagger2是第一个实现生成代码的完整堆栈框架。生成代码的指导原则是模仿用户可能的手写的代码来确保依赖注入简单，可跟踪以及性能。

##Using Dagger
我们通过构建一个咖啡机来示例依赖注入和Dagger的使用。[完整代码链接](https://github.com/google/dagger/tree/master/examples/simple/src/main/java/coffee)。

###Declaring Dependencies
Dagger构建应用类的实例并满足它们的依赖。它使用javax.inject.Inject注解来确定哪些构造器和字段是它感兴趣的。

使用@Inject注解构造器，Dagger就会使用这个构造器来创建类实例。当需要一个新实例时，Dagger会获取需要的参数并调用这个构造器。

```
class Thermosiphon implements Pump {
  private final Heater heater;

  @Inject
  Thermosiphon(Heater heater) {
    this.heater = heater;
  }

  ...
}
```

Dagger可以直接注入字段。在下面的例子中，Dagger为heater字段获取一个Heater实例以及为pump字段获取一个Pump实例。

```
class CoffeeMaker {
  @Inject Heater heater;
  @Inject Pump pump;

  ...
}
```

如果你的类有@Inject注解的字段但是没有@Inject注解的构造器，Dagger会在需要的时候注解这些字段，但是不会创建新实例。添加一个@Inject注解的无参数构造器来标识Dagger也可能创建实例。

虽然构造器和字段通常是首选的，但Dagger也支持方法注入。

没有@Inject注解的类不会被Dagger构建。

###Satisfying Dependencies
默认情况下，Dagger通过构建一个上面描述所需要的类型实例来满足每个依赖。当你需要一个CoffeeMaker，它会调用new CoffeeMaker()获取一个实例并设置注入的字段。

但@inject并不是哪里都可以工作：

 - 不能构建接口。
 - 不能注解第三方类。
 - 可配置对象必须已配置。

对于这些情形，@Inject是不能够满足的，可以使用@Provides注解方法来满足依赖。这个方法返回类型标识它满足哪种依赖。

例如，当需要一个Heater时，provideHeater()会调用。

```
@Provides static Heater provideHeater() {
  return new ElectricHeater();
}
```

@Provides方法也可能有它们自己的依赖。下面这个会当需要一个Pump时返回一个Thermosiphon：

```
@Provides static Pump providePump(Thermosiphon pump) {
  return pump;
}
```

所有的@Provides方法必须归属于一个module，module就是被@Module注解的类。

```
@Module
class DripCoffeeModule {
  @Provides static Heater provideHeater() {
    return new ElectricHeater();
  }

  @Provides static Pump providePump(Thermosiphon pump) {
    return pump;
  }
}
```

按照惯例，@Provides方法都以provide前缀命名，而module都以Module后缀命名。

###Building the Graph
@Inject和@Provides注解的类通过它们的依赖链接起来，形成了一个对象图。调用代码像应用中的main方法或者Android的Application通过一组良好定义的根集来访问对象图。这些集通过一个接口定义，接口中的方法没有参数，并返回期望的类型。通过应用@Component注解到这样的接口上，并传递module类型到modules参数，Dagger2然后会生成这个接口的实现。

```
@Component(modules = DripCoffeeModule.class)
interface CoffeeShop {
  CoffeeMaker maker();
}
```

实现与接口名字相同并且以Dagger为前缀。通过在这个实现上调用builder()方法获取实例，使用返回的builder来设置依赖以及调用build()方法获取一个新实例。

```
CoffeeShop coffeeShop = DaggerCoffeeShop.builder()
    .dripCoffeeModule(new DripCoffeeModule())
    .build();
```

如果你的@Component不是一个顶级类型，那么生成的component名称会以下划线连接来包括它的闭包类型名称。例如下面的代码生成的component名称为DaggerFoo_Bar_BazComponent。

```
class Foo {
  static class Bar {
    @Component
    interface BazComponent {}
  }
}
```

如果一个module它所有的@provides方法都是静态的，那么接口实现就会有一个create()方法，而无需进行build直接可以通过这个方法获取一个新实例。

```
CoffeeShop coffeeShop = DaggerCoffeeShop.create();
```

###Singletons and Scoped Bindings
用@Singleton注解@Provides方法或可注入类，对象图就会在所有的客户端使用一个实例。

```
@Provides @Singleton static Heater provideHeater() {
  return new ElectricHeater();
}
```

用@Singleton注解可注入类，提醒潜在的维护人员这个类可能被多个线程共享。

```
@Singleton
class CoffeeMaker {
  ...
}
```

因为Dagger2在对象图中关联scope实例与component实现实例，components需要声明他们想要存在的scope。例如，不会让一个@Singleton绑定和一个@RequestScoped绑定存在一个component，因为它们的scope有不同的生命周期，因此必须存在于拥有不同生命周期的组件中。应用scope注解在component接口上，就可以声明coponent关联给定的scope。

```
@Component(modules = DripCoffeeModule.class)
@Singleton
interface CoffeeShop {
  CoffeeMaker maker();
}
```

一个component可能应用多个scope注解，这表明它们是相同scope的别名，因此component可以包含它声明的所有scope绑定。

###Reusable scope
有时你希望限制@Inject构造器类或者@Provides方法的调用次数，但是你不需要确保在特定component或subcomponent使用相同的实例。这在像Android这种分配内存昂贵的环境下很有用。

对于这些绑定，你可以应用@Reusable scope，@Reusable绑定，不像其他scope，不会与任何一个component关联，而每个使用这个绑定的component都会缓存返回的实例或初始化对象。

这意味着如果你在一个component中安装一个包含@Reusable绑定的module，但是只有一个subcomponent使用了这个绑定，那么只有这个subcomponent会缓存这个绑定对象。如果两个subcomponent没有共享父类，每个都使用了这个绑定，那么它们都会缓存各自的对象。如果component的父类已经缓存了这个对象，那么subcomponent会复用。

不会确保component只调用一次绑定，因此应用@Reusable绑定返回易变对象或需要引用相同实例的对象是危险的。只有对于那些不变对象才是安全的。如果你不关心分配多少对象，你可以不使用scope。

```
@Reusable // It doesn't matter how many scoopers we use, but don't waste them.
class CoffeeScooper {
  @Inject CoffeeScooper() {}
}

@Module
class CashRegisterModule {
  @Provides
  @Reusable // DON'T DO THIS! You do care which register you put your cash in.
            // Use a specific scope instead.
  static CashRegister badIdeaCashRegister() {
    return new CashRegister();
  }
}

@Reusable // DON'T DO THIS! You really do want a new filter each time, so this
          // should be unscoped.
class CoffeeFilter {
  @Inject CoffeeFilter() {}
}
```

###Releasable references
当一个绑定使用了scope注解，意味着component对象持有绑定对象的引用直到component对象被垃圾回收。在内存敏感环境例如Android，你可能希望当应用内存紧张时当前没有使用的scoped对象在垃圾回收时被删除。

这种情况下，你可以定义一个scope并用@CanReleaseReferences注解。

```
@Documented
@Retention(RUNTIME)
@CanReleaseReferences
@Scope
public @interface MyScope {}
```

当你检测到你希望允许当前scope持有的、没有被其他对象使用的对象可以在垃圾时被删除，你可以为你的scope注入一个ReleasableReferenceManager对象并调用releaseStrongReferences()，这样会让component持有对象的一个WeakReference而不是强引用。

```
@Inject @ForReleasableReferences(MyScope.class)
ReleasableReferences myScopeReferences;

void lowMemory() {
  myScopeReferences.releaseStrongReferences();
}
```

如果你检测到内存压力减小时，你可以通过调用restoreStrongReferences()恢复没有在垃圾回收时删除的缓存对象的强引用：

```
void highMemory() {
  myScopeReferences.restoreStrongReferences();
}
```

###Lazy injections
有时你需要一个对象延迟初始化。对于绑定对象T，你可以创建一个Lazy&lt;T&gt;来推迟初始化直到第一次调用Lazy&lt;T&gt;的get()方法。如果T是singleton，那么Lazy&lt;T&gt;也应该在整个对象图注入中是一个实例。否则，每个注入点都会得到自己的Lazy&lt;T&gt;实例。无论如何，任何Lazy&lt;T&gt;实例的后续调用都会返回相同的T实例。

```
class GridingCoffeeMaker {
  @Inject Lazy<Grinder> lazyGrinder;

  public void brew() {
    while (needsGrinding()) {
      // Grinder created once on first call to .get() and cached.
      lazyGrinder.get().grind();
    }
  }
}
```

###Provider injections
有时你需要返回多个实例而不是仅仅注入一个值。你有多种选择(Factories，Builders等等)，一个选择就是注入一个Provider&lt;T&gt;而不是T。Provider&lt;T&gt;在每次调用get()时都会调用T的绑定逻辑。如果T的绑定个逻辑是一个@Inject构造器，那么就会创建一个新实例，但是@Providers方法并不保证。

###Qualifiers
有时候只有类型不足够标识一个依赖。例如，在一个复杂的咖啡机应用可能希望区分water和hot plate的加热器。

在这种情形下，我们添加一个qualifier注解。这个注解本身被@Qualifier注解。下面是javax.inject中的qualifier注解@Named的声明：

```
@Qualifier
@Documented
@Retention(RUNTIME)
public @interface Named {
  String value() default "";
}
```

你可以创建自己的qualifier注解，或者直接使用@Named。在感兴趣的字段或参数上应用qulifier注解。类型和qualifier注解会同时用来标识依赖。

```
class ExpensiveCoffeeMaker {
  @Inject @Named("water") Heater waterHeater;
  @Inject @Named("hot plate") Heater hotPlateHeater;
  ...
}
```

在相应的@Provides方法上通过注解提供qualified值。

```
@Provides @Named("hot plate") static Heater provideHotPlateHeater() {
  return new ElectricHeater(70);
}

@Provides @Named("water") static Heater provideWaterHeater() {
  return new ElectricHeater(93);
}
```

依赖不能有多个qualifier注解。

###Optional bindings
如果你希望即使组件中一些依赖尚未绑定但仍可工作，你可以在module添加一个@BindsOptionalOf方法。

```
@BindsOptionalOf abstract CoffeeCozy optionalCozy();
```

这意味着@Inject构造器、字段以及@Provides方法可以依赖一个Optional&lt;CoffeeCozy&gt;对象。如果在component中有一个CoffeeCozy绑定，那么Optional就是存在的，如果没有CoffeeCozy绑定，则Optional就是不存在的。

###Binding Instances
通常在你构建component时，你已经有可用数据了。例如，假如你有一个应用使用命令行参数，你可能希望在你的component中绑定这些参数。

也许你的应用使用一个参数来代表用户名称，你想注入到@UserName字符串。你可以在component builder中添加一个方法，并用@BindsInstance来允许实例注入到component中。

```
@Component(modules = AppModule.class)
interface AppComponent {
  App app();

  @Component.Builder
  interface Builder {
    @BindsInstance Builder userName(@UserName String userName);
    AppComponent build();
  }
}
```

你的应用可能看起来像这样：

```
public static void main(String[] args) {
  if (args.length > 1) { exit(1); }
  App app = DaggerAppComponent
      .builder()
      .userName(args[0])
      .build()
      .app();
  app.run();
}
```

在上面的例子中，当调用username()这个方法时，component中的@UserName会使用传递给Builder的实例。在构建component之前，所有的@BindsInstance方法都必须调用，并传递非空值。

如果@BindsInstance方法的参数标记为@Nullable，那么绑定以及@Provider方法也被认为"nullable"：注入端必须也标记为@Nullable，在绑定时null是可接受的。更多的，Builder用户可能忘记调用这个方法，而component会视这个实例为null。

@BindsInstance方法应该优先于@Module使用构造器参数以及立即传值。

###Compile-time Validation
Dagger注解处理器是严格的，如果绑定非法或者未完成就会触发编译错误。例如，一个component中安装了下面这个丢失了Executor的绑定的module：

```
@Module
class DripCoffeeModule {
  @Provides static Heater provideHeater(Executor executor) {
    return new CpuHeater(executor);
  }
}
```

当编译时，javac就会拒绝丢失绑定：

```
[ERROR] COMPILATION ERROR :
[ERROR] error: java.util.concurrent.Executor cannot be provided without an @Provides-annotated method.
```

通过在这个component中的任何一个module中为Executor添加一个@Provides注解方法来修复。因为@Inject，@Module和@Provides注解都是分开验证的，因此绑定关系的所有验证发生在@Component级别。

###Compile-time Code Generation
Dagger的注解处理器可能生成像CoffeeMaker_Factory.java或者CoffeMaker_MembersInjector.java这样名字的源文件。这些文件是Dagger的实现细节。你不应该直接使用它们，你可以在代码中唯一可以引用的是针对component的以Dagger为前缀的类型。