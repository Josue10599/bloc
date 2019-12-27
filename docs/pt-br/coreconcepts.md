# Principais Conceitos

?> Leia e compreenda cuidadosamente as seções a seguir antes de trabalhar com o [bloc](https://pub.dev/packages/bloc).

Existem vários conceitos essenciais que são fundamentais para entender como usar o Bloc.

Nas próximas seções, discutiremos cada uma delas em detalhes, bem como trabalharemos como elas se aplicariam a um aplicativo do mundo real: um contador app.

## Eventos

> Eventos são a entrada para um bloco. Eles são geralmente adicionados em resposta a interações do usuário, como pressionar botões ou eventos do ciclo de vida, como carregamentos de páginas.

Ao projetar um aplicativo, precisamos dar um passo atrás e definir como o usuário irá interagir com ele. No contexto do nosso aplicativo contador, teremos dois botões um para aumentar e outro para diminuir o valor.

Quando um usuário toca em um desses botões, algo precisa acontecer para notificar o "cérebro" do nosso aplicativo, para que ele possa responder à ação do usuário; é nesta situação que os eventos entram em cena.

Precisamos ser capazes de notificar o "cérebro" da nossa aplicação sobre um incremento e um decremento, portanto precisamos definir esses eventos.

```dart
enum CounterEvent { increment, decrement }
```

Nesse caso, podemos representar os eventos usando um, `enum` mas para casos mais complexos, pode ser necessário usar um `class`, especialmente se for necessário passar informações ao bloco.

Neste ponto, definimos o nosso primeiro evento! Observe que não usamos o Bloc de nenhuma maneira até agora e não há mágica acontecendo; é apenas código Dart simples.

## Estados

> Os estados são a saída de um bloco e representam uma parte do estado do aplicativo. Os componentes da interface do usuário podem ser notificados sobre o estados e redesenhar partes deles com base no estado atual.

Até o momento, foram definidos dois eventos aos quais nosso aplicativo responderá: `CounterEvent.increment` e `CounterEvent.decrement`.

Agora precisamos definir como representar o estado do nosso aplicativo.

Como estamos construindo apenas um contador, nosso estado é muito simples: é apenas um número inteiro que representa o valor atual do contador.

Veremos exemplos mais complexos de estado posteriormente, mas neste caso, um tipo primitivo é perfeitamente adequado como representação de estado.

## Transições

> A mudança de um estado para outro recebe o nome de transição. Uma transição consiste no estado atual, no evento e no próximo estado.

À medida que o usuário interage com nosso aplicativo de contador, ele dispara Increment e Decrement eventos que atualizam o estado do contador. Todas essas alterações de estado podem ser descritas como uma série de `Transitions`.

Por exemplo, se um usuário abrisse nosso aplicativo e tocasse no botão de incremento, veríamos a seguinte `Transition`.

```json
{
  "currentState": 0,
  "event": "CounterEvent.increment",
  "nextState": 1
}
```

Como todas as alterações de estado são registadas, somos capazes de instrumentar com facilidade nosso aplicativo e rastrear todas as interações do usuário e alterações de estado em um único local. Além disso, isso possibilita coisas como depuração de viagens no tempo.

## Streams

?> Confira a documentação oficial [Dart Documentation] (https://www.dartlang.org/tutorials/language/streams) para obter mais informações sobre o `Streams`.

> Um Stream é uma sequência de dados assíncronos.

O Bloc é construído sobre o [RxDart](https://pub.dev/packages/rxdart); no entanto, abstrai todos os detalhes específicos da implementação do `RxDart`.

Para usar o Bloc, é essencial ter uma sólida compreensão dos `Streams` e como eles funcionam.

> Se você não estiver familizarizado com os Streams, vamos fazer uma abstração, pense em um cano com água fluindo através dele. O tubo é o Stream e a água são os dados assíncronos.

Podemos criar um `Stream` no Dart escrevendo uma função `async*`.

```dart
Stream<int> countStream(int max) async* {
    for (int i = 0; i < max; i++) {
        yield i;
    }
}
```

Marcando uma função como `async*` podemos usar a palavra-chave `yield ` e retornar um dado `Stream`. No exemplo acima, estamos retornando um número inteiro até o parâmetro `max`.

Toda vez que utilizamos o `yield ` em uma função  `async*`, estamos enviando esse dados através do `Stream`.

Podemos consumir o `Stream` exposto acima de várias maneiras. Se quiséssemos escrever uma função para retornar a soma de um número inteiro pela `Stream`, poderia ser algo como: 

```dart
Future<int> sumStream(Stream<int> stream) async {
    int sum = 0;
    await for (int value in stream) {
        sum += value;
    }
    return sum;
}
```

Marcando a função acima com `async` podemos usar a palavra-chave `await ` e retornar um número inteiro pelo `Future`. Neste exemplo, estamos aguardando cada valor no fluxo e retornando a soma de todos os números inteiros no fluxo.

Podemos juntar tudo assim:

```dart
void main() async {
    /// Initialize a stream of integers 0-9
    Stream<int> stream = countStream(10);
    /// Compute the sum of the stream of integers
    int sum = await sumStream(stream);
    /// Print the sum
    print(sum); // 45
}
```

## Blocs

> A Bloc (Business Logic Component) is a component which converts a `Stream` of incoming `Events` into a `Stream` of outgoing `States`. Think of a Bloc as being the "brains" described above.

> Every Bloc must extend the base `Bloc` class which is part of the core bloc package.

```dart
import 'package:bloc/bloc.dart';

class CounterBloc extends Bloc<CounterEvent, int> {

}
```

In the above code snippet, we are declaring our `CounterBloc` as a Bloc which converts `CounterEvents` into `ints`.

> Every Bloc must define an initial state which is the state before any events have been received.

In this case, we want our counter to start at `0`.

```dart
@override
int get initialState => 0;
```

> Every Bloc must implement a function called `mapEventToState`. The function takes the incoming `event` as an argument and must return a `Stream` of new `states` which is consumed by the presentation layer. We can access the current bloc state at any time using the `state` property.

```dart
@override
Stream<int> mapEventToState(CounterEvent event) async* {
    switch (event) {
      case CounterEvent.decrement:
        yield state - 1;
        break;
      case CounterEvent.increment:
        yield state + 1;
        break;
    }
}
```

At this point, we have a fully functioning `CounterBloc`.

```dart
import 'package:bloc/bloc.dart';

enum CounterEvent { increment, decrement }

class CounterBloc extends Bloc<CounterEvent, int> {
  @override
  int get initialState => 0;

  @override
  Stream<int> mapEventToState(CounterEvent event) async* {
    switch (event) {
      case CounterEvent.decrement:
        yield state - 1;
        break;
      case CounterEvent.increment:
        yield state + 1;
        break;
    }
  }
}
```

!> Blocs will ignore duplicate states. If a Bloc yields `State nextState` where `state == nextState`, then no transition will occur and no change will be made to the `Stream<State>`.

At this point, you're probably wondering _"How do I notify a Bloc of an event?"_.

> Every Bloc has a `add` method. `Add` takes an `event` and triggers `mapEventToState`. `Add` may be called from the presentation layer or from within the Bloc and notifies the Bloc of a new `event`.

We can create a simple application which counts from 0 to 3.

```dart
void main() {
    CounterBloc bloc = CounterBloc();

    for (int i = 0; i < 3; i++) {
        bloc.add(CounterEvent.increment);
    }
}
```

!> By default, events will always be processed in the order in which they were added and any newly added events are enqueued. An event is considered fully processed once `mapEventToState` has finished executing.

The `Transitions` in the above code snippet would be

```json
{
    "currentState": 0,
    "event": "CounterEvent.increment",
    "nextState": 1
}
{
    "currentState": 1,
    "event": "CounterEvent.increment",
    "nextState": 2
}
{
    "currentState": 2,
    "event": "CounterEvent.increment",
    "nextState": 3
}
```

Unfortunately, in the current state we won't be able to see any of these transitions unless we override `onTransition`.

> `onTransition` is a method that can be overridden to handle every local Bloc `Transition`. `onTransition` is called just before a Bloc's `state` has been updated.

?> **Tip**: `onTransition` is a great place to add bloc-specific logging/analytics.

```dart
@override
void onTransition(Transition<CounterEvent, int> transition) {
    print(transition);
}
```

Now that we've overridden `onTransition` we can do whatever we'd like whenever a `Transition` occurs.

Just like we can handle `Transitions` at the bloc level, we can also handle `Exceptions`.

> `onError` is a method that can be overriden to handle every local Bloc `Exception`. By default all exceptions will be ignored and `Bloc` functionality will be unaffected.

?> **Note**: The stacktrace argument may be `null` if the state stream received an error without a `StackTrace`.

?> **Tip**: `onError` is a great place to add bloc-specific error handling.

```dart
@override
void onError(Object error, StackTrace stackTrace) {
  print('$error, $stackTrace');
}
```

Now that we've overridden `onError` we can do whatever we'd like whenever an `Exception` is thrown.

## BlocDelegate

One added bonus of using Bloc is that we can have access to all `Transitions` in one place. Even though in this application we only have one Bloc, it's fairly common in larger applications to have many Blocs managing different parts of the application's state.

If we want to be able to do something in response to all `Transitions` we can simply create our own `BlocDelegate`.

```dart
class SimpleBlocDelegate extends BlocDelegate {
  @override
  void onTransition(Bloc bloc, Transition transition) {
    super.onTransition(bloc, transition);
    print(transition);
  }
}
```

?> **Note**: All we need to do is extend `BlocDelegate` and override the `onTransition` method.

In order to tell Bloc to use our `SimpleBlocDelegate`, we just need to tweak our `main` function.

```dart
void main() {
  BlocSupervisor.delegate = SimpleBlocDelegate();
  CounterBloc bloc = CounterBloc();

  for (int i = 0; i < 3; i++) {
    bloc.add(CounterEvent.increment);
  }
}
```

If we want to be able to do something in response to all `Events` added, we can also override the `onEvent` method in our `SimpleBlocDelegate`.

```dart
class SimpleBlocDelegate extends BlocDelegate {
  @override
  void onEvent(Bloc bloc, Object event) {
    super.onEvent(bloc, event);
    print(event);
  }

  @override
  void onTransition(Bloc bloc, Transition transition) {
    super.onTransition(bloc, transition);
    print(transition);
  }
}
```

If we want to be able to do something in response to all `Exceptions` thrown in a Bloc, we can also override the `onError` method in our `SimpleBlocDelegate`.

```dart
class SimpleBlocDelegate extends BlocDelegate {
  @override
  void onEvent(Bloc bloc, Object event) {
    super.onEvent(bloc, event);
    print(event);
  }

  @override
  void onTransition(Bloc bloc, Transition transition) {
    super.onTransition(bloc, transition);
    print(transition);
  }

  @override
  void onError(Bloc bloc, Object error, StackTrace stacktrace) {
    super.onError(bloc, error, stacktrace);
    print('$error, $stacktrace');
  }
}
```

?> **Note**: `BlocSupervisor` is a singleton which oversees all Blocs and delegates responsibilities to the `BlocDelegate`.
