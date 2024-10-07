# Riverpod in Flutter Development

Riverpod is a state management solution for Flutter that stands out due to its robustness, flexibility, and enhanced performance over traditional options like `Provider`. Here’s a detailed analysis covering its advantages, disadvantages, alternatives, implementation examples, migration strategies, and best practices.

ProviderScope :- It keep track  of all the provider and see that if there is no leak state
```dart
void main() {
  runApp(
    ProviderScope( // Root of all providers
      child: MyApp(),
    ),
  );
}
```
# Provider

1. Provider
    1. This is object that provide the data to widget or other provider
    2. it is read only widget and cannot update it value
    
    ```dart
    
    final nameProvider = Provider<dataType>((ref) {
      return dataType_Value
    });
    ```
    
    1. it use to provide primitive , non primitive and instance of classes
    2. there are 2 way to read
    3. read: Allows a provider to access the value stored in another provider, making it available for use within the current provider.
        1. Use of Consumer Widget
        ![image](https://github.com/user-attachments/assets/a5c38739-5a1b-4134-bb93-6b3b597e408d)
        
        This is the file where i want to display value store in provider ,i have converted it into consumer widget
        
     WidgetRef is a specialized version of Ref used in widgets, allowing widgets to interact directly with providers. It can trigger widget rebuilds whenever the state of a watched provider changes, enabling real-time updates to the UI based on provider changes
        

## There are 3 method to call the data from the provider

1. `ref.watch(nameProvider)`
    
    it is recommended to use watch inside the build function
    
    ```dart
    class HomeScreen extends ConsumerWidget {
      const HomeScreen({super.key});
    
      @override
      Widget build(BuildContext context, WidgetRef ref) {
        final name = ref.watch(nameProvider);
        return Scaffold(
          body: Center(
            child: Text(
              name,
              style: const TextStyle(fontSize: 24.0),
            ),
          ),
        );
      }
    }
    ```
    
2. Consumer Builder
    
    1. This method is similiar but here we do not convert stateless widget to Consumer widget
    2. We wrap the widget with consumer Builder widget 
    3. this help when we want to target the specfic area of screen 
    but suppose if we want to display name in appbar then it would be difficult 
    4. because we have to wrap widget again with consumer widget
    ![image (1)](https://github.com/user-attachments/assets/6cb20f58-3182-474f-b9dc-3726c3447ba8)

3.  `ref.read(nameProvider)`
    1. it use in lifecycle method
    2. function we create
    3. This is the one time thing
    4. we use watch when we there is continuous change value of provider

## Consumer State

This state come with or we declare in ConsumerStatefull widget 

it consist of inbuild ref

so we don’t have to use Consumer Builder and also Widget ref

# State Provider

This provider help us to update the value of the provider from outside of file which is not possible in normal provider

```dart
//-- State Provider

final nameProvider = StateProvider<String?>((ref) => null);
this provider tell that i can be null and also my return type is string

here .notifier help us to update the value also it convert the string return type to 
a controller
ref.read(nameProvider.notifier);

:- StateController<String?> Function(ProviderListenable<StateController<String?>>)

.state tell us the return type of Provider
***ref.read(nameProvider.notifier).state;***

T get state
Type: String?

```

### this code tell us how to update the value of provider outside the file

```dart
// home_screen.dart
class HomeScreen extends ConsumerWidget {
  const HomeScreen({super.key});

  void onSubmit(WidgetRef ref, String value) {
    ref.read(nameProvider.notifier).update((state) => value);
  }

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final name = ref.watch(nameProvider) ?? "";
    return Scaffold(
      body: Column(
        children: [
          Text(name),
          Center(
            child: TextField(onSubmitted: (value) => onSubmit(ref, value)),
          ),
        ],
      ),
    );
  }
}
```

# State Notifier and State Notifier Provider

This is the better method than StateProvider 

my user model

```dart
class UserModel {
  final String name;
  final int age;
  UserModel({
    required this.name,
    required this.age,
  });
}

```

this class will help us to update the datatype which we pass
`class UserNotifier extends StateNotifier<datatype> {}`

in our case we want to update userModel
`class UserNotifier extends StateNotifier<UserModel> {}`

```dart
 
class UserNotifier extends StateNotifier<UserModel> {
  UserNotifier(super.state);

  void updateName(String newName) {
    // name = newName;                //! cannot do like this
    // state.name = state.newName      //! cannot do like this

    //-- One way is
    state = UserModel(name: newName, age: state.age); //?? this will work but not for bigger code

    // we can use of the method like fromMap , copyMap that we have generated from data class
    
  }
```

```dart
class UserModel {
  final String name;
  final int age;
  const UserModel({
    required this.name,
    required this.age,
  });

  UserModel copyWith({
    String? name,
    int? age,
  }) {
    return UserModel(
      name: name ?? this.name,
      age: age ?? this.age,
    );
  }

  Map<String, dynamic> toMap() {
    return <String, dynamic>{
      'name': name,
      'age': age,
    };
  }

  factory UserModel.fromMap(Map<String, dynamic> map) {
    return UserModel(
      name: map['name'] as String,
      age: map['age'] as int,
    );
  }

  String toJson() => json.encode(toMap());

  factory UserModel.fromJson(String source) => UserModel.fromMap(json.decode(source) as Map<String, dynamic>);

  @override
  String toString() => 'UserModel(name: $name, age: $age)';

  @override
  bool operator ==(covariant UserModel other) {
    if (identical(this, other)) return true;

    return other.name == name && other.age == age;
  }

  @override
  int get hashCode => name.hashCode ^ age.hashCode;
}

=========================================================================================

class UserNotifier extends StateNotifier<UserModel> {
  UserNotifier(super.state);

  void updateName(String newName) {
    //! we can use of the method like fromMap , copyMap that we have generated from data class
    state = state.copyWith(name: newName);
  }
}

```

now to expose this statenotifier we use Provider state Notifier

```dart
final userProvider = StateNotifierProvider((ref) => UserNotifier(const UserModel(age: 20, name: "ramesh")));
StateNotifierProvider this required stateNotifier class along with initial state
```

```dart

class HomeScreen extends ConsumerWidget {
  const HomeScreen({super.key});
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final user = ref.watch(userProvider);
    return Scaffold(
      body: Column(
        children: [
          Text(user.name), // this will give error
          This will give an error because the variable user doesn’t know that it is of type UserModel
        ],
      ),
    );
  }
}
one way to fix it 
final user = ref.watch(userProvider) as UserModel; 

but this line of code we have to write every where in the file

```

Fix the issue

```dart
                                        // class we are  this is the 
                                        // returing     state of the
                                                        //class
final userProvider = StateNotifierProvider<UserNotifier, UserModel>((ref) => UserNotifier(const UserModel(age: 20, name: "ramesh")));

```
![image (1)](https://github.com/user-attachments/assets/e8a4249f-ff36-467b-99d6-8cb56cd6816f)
initial screen

```dart
  void onSubmit(WidgetRef ref, String value) {
    final user = ref.read(userProvider.notifier).updateName(value);
  }
  this create the instance of UserNotifier class and then we can access it function
  

```
### If we want to re-run only when the `name` value changes, we can use a `.select` to listen specifically to `name`:
---


```dart
final userName = ref.watch(userProvider.select((value) => value.name));
```

---

This approach watches only `name` within `userProvider`, so the widget rebuilds only when `name` changes, rather than any other properties of `userProvider`.

# FutureProvider and StreamProvider in Flutter

## FutureProvider

FutureProvider is used when you need to fetch data asynchronously, like from an API or a database, and return a single result. It automatically listens for the Future and updates the UI when the data is available.

In this example, we use a FutureProvider to fetch a single user's information from an HTTP endpoint.

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:http/http.dart' as http;

// Define the FutureProvider for fetching user data asynchronously
final fetchUserProvider = FutureProvider<UserModel>((ref) async {
  const uri = "<https://jsonplaceholder.typicode.com/users/1>";
  final response = await http.get(Uri.parse(uri));
  return UserModel.fromJson(response.body);
});

```

### UserModel Definition

The `UserModel` class represents the user data model to map the fetched JSON data.

```dart
class UserModel {
  final String name;
  final String email;

  UserModel({
    required this.name,
    required this.email,
  });

  // Factory method to create a UserModel from JSON
  factory UserModel.fromJson(String body) {
    final data = jsonDecode(body);
    return UserModel(
      name: data['name'],
      email: data['email'],
    );
  }
}

```

### Explanation

- **`fetchUserProvider`**: This FutureProvider fetches data from the API.
- **Request (req)**: Obtains information from the client or API.
- **Response (res)**: Sends back the requested data.

---

## StreamProvider

StreamProvider is used for continuous data updates, like a live data feed, and is ideal for situations where you need real-time updates (e.g., chat messages, notifications).

### Example of StreamProvider

```dart
import 'dart:async';
import 'dart:math';
import 'package:flutter_riverpod/flutter_riverpod.dart';

// StreamProvider to simulate a live feed of random numbers
final randomNumberProvider = StreamProvider<int>((ref) {
  return Stream.periodic(
    Duration(seconds: 1),
    (_) => Random().nextInt(100),
  );
});

```

### Explanation

- **`randomNumberProvider`**: This StreamProvider generates a new random number every second.
- **Stream.periodic**: Emits a new random number every second, simulating a real-time data feed.

---

**Usage**: In your widget, you can listen to `fetchUserProvider` and `randomNumberProvider` to update the UI based on the fetched data.

```dart
Widget build(BuildContext context, WidgetRef ref) {
  // Access FutureProvider
  final userAsyncValue = ref.watch(fetchUserProvider);

  // Access StreamProvider
  final randomNumAsyncValue = ref.watch(randomNumberProvider);

  return Column(
    children: [
      // Displaying user data from FutureProvider
      userAsyncValue.when(
        data: (user) => Text('User: ${user.name}, Email: ${user.email}'),
        loading: () => CircularProgressIndicator(),
        error: (error, stack) => Text('Error: $error'),
      ),

      // Displaying live random number from StreamProvider
      randomNumAsyncValue.when(
        data: (randomNum) => Text('Random Number: $randomNum'),
        loading: () => CircularProgressIndicator(),
        error: (error, stack) => Text('Error: $error'),
      ),
    ],
  );
}

```

---


## Advantages of Riverpod

1. **Type Safety**:  
   Riverpod provides compile-time safety, allowing for early error detection and preventing runtime errors related to provider access. This type safety facilitates easier refactoring, aided by strong IDE support.

2. **Testing**:  
   Riverpod promotes a highly testable codebase due to its immutable state. Mocking providers becomes straightforward, making unit and widget testing easier.
   
3. **Performance**:  
   Riverpod optimizes performance with fine-grained rebuilds and efficient dependency tracking. Memory management is improved through auto-disposal, leading to efficient resource use.

## Disadvantages

1. **Learning Curve**:  
   For beginners, Riverpod can present a steep learning curve due to its complex syntax and the need to understand various provider types (e.g., `StateProvider`, `StateNotifierProvider`). A solid grasp of reactive programming is also essential.

2. **Boilerplate Code**:  
   Riverpod requires more initial setup compared to simpler solutions, including creating provider definitions and additional code for provider combinations.

3. **Migration Challenges**:  
   Gradually adopting Riverpod in existing projects can be difficult, often necessitating significant refactoring. Teams need to familiarize themselves with new patterns.

## Alternatives to Riverpod

1. **Provider**:  
   A simpler, more basic state management solution ideal for small applications. It’s easier to learn but lacks the power and flexibility of Riverpod.

2. **Bloc**:  
   A more structured approach suitable for large applications, offering strong separation of concerns but being more verbose than Riverpod.

3. **GetX**:  
   Known for its simplicity and all-in-one solution approach, GetX offers less type safety and might result in less maintainable code.

## When to Use Riverpod

### Best For:
- Medium to large applications
- Teams prioritizing type safety
- Projects requiring complex state management
- Applications needing extensive testing

### Maybe Not For:
- Small, simple applications
- Teams new to Flutter
- Quick prototypes
- Projects with tight deadlines and learning constraints


### Conclusion

In summary, Riverpod is an excellent choice for managing state in Flutter applications, especially when prioritizing type safety and testing. While it may have a learning curve and introduce additional boilerplate code, its advantages in code organization, performance, and developer experience make it a powerful tool for medium to large applications.
