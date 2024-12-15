# Alpena

A lightweight, dependency-aware reactive data structure for JavaScript applications.

## Table of Contents

- [Introduction](#introduction)
- [Features](#features)
- [Installation](#installation)
- [Getting Started](#getting-started)
- [API Reference](#api-reference)
  - [Constructor](#constructor)
  - [Properties](#properties)
  - [Methods](#methods)
    - [subscribe(callback)](#subscribecallback)
    - [unsubscribe(callback)](#unsubscribecallback)
    - [addDependency(...dependencies)](#adddependencydependencies)
    - [removeDependency(dependency)](#removedependencydependency)
    - [removeAllDependencies()](#removealldependencies)
    - [update(updateFunction)](#updateupdatefunction)
    - [get()](#get)
    - [set(value)](#setvalue)
- [Examples](#examples)
  - [Basic Usage](#basic-usage)
  - [Dependency Management](#dependency-management)
  - [Working with Arrays and Objects](#working-with-arrays-and-objects)
- [Utilities and Helpers](#utilities-and-helpers)
- [Best Practices](#best-practices)

## Introduction

**Alpena** is a reactive data structure designed to simplify state management in JavaScript applications. It allows you to create data signals that can be subscribed to, and automatically notifies subscribers when the signal's value changes. Additionally, signals can have dependencies on other signals, creating a reactive network where changes propagate efficiently.

## Features

- **Reactive Data Handling**: Subscribe to changes in data and react accordingly.
- **Dependency Management**: Define dependencies between signals to create reactive data flows.
- **Immutable Updates**: Ensure data integrity with deep equality checks.
- **Type Safety**: Prevent unintended type changes with type checking on updates.
- **Convenience Methods**: Utility methods like `update` for ease of use.

## Installation

You can include Alpena in your project by importing the class directly. Since this is a standalone class, there is no need for installation via npm. Just ensure the class definition is included in your project.

```javascript
// Ensure this import path matches where Alpena is located in your project.
import Alpena from './Alpena.js';
```

## Getting Started

Begin by creating instances of `Alpena` and subscribing to changes. You can also define dependencies between signals to create more complex reactive relationships.

```javascript
import Alpena from './Alpena.js';
const A = new Alpena(1);
const B = new Alpena('ABC');

B.addDependency(A);
B.subscribe((v, a)=>console.log('RESULT', v,a))

// when A changes B prints
// RESULT ABC 1
// RESULT ABC 2
// RESULT ABC 3
// RESULT ABC 4

setInterval(function () {
  A.update(v => v+1);
}, 1_000)

```

## API Reference

### Constructor

#### `new Alpena(initialValue)`

- **Parameters:**
  - `initialValue`: The initial value of the signal.

- **Usage:**

  ```javascript
  const signal = new Alpena(100);
  ```

### Properties

#### `value`

- **Description:** Getter and setter for the signal's value.

- **Usage:**

  ```javascript
  const signal = new Alpena('Hello');

  // Get value
  console.log(signal.value); // Output: 'Hello'

  // Set value
  signal.value = 'World';
  ```

### Methods

#### `subscribe(callback)`

- **Description:** Subscribe to changes in the signal's value. The `callback` is invoked immediately with the current value upon subscription and subsequently whenever the value changes.

- **Parameters:**
  - `callback(value, ...dependencyValues)`: A function executed when the signal's value changes. Receives the current signal value and values of its dependencies.

- **Returns:** A function that can be called to unsubscribe.

- **Usage:**

  ```javascript
  const unsubscribe = signal.subscribe((value, dep1, dep2) => {
    console.log(`Signal value: ${value}`);
  });

  // To unsubscribe
  unsubscribe();
  ```

#### `unsubscribe(callback)`

- **Description:** Unsubscribe a previously subscribed callback.

- **Parameters:**
  - `callback`: The callback function to remove from subscribers.

- **Usage:**

  ```javascript
  signal.unsubscribe(myCallback);
  ```

#### `addDependency(...dependencies)`

- **Description:** Add one or more `Alpena` instances as dependencies. When a dependency changes, the signal's subscribers are notified.

- **Parameters:**
  - `dependencies`: One or more `Alpena` instances to add as dependencies.

- **Usage:**

  ```javascript
  const signalA = new Alpena(1);
  const signalB = new Alpena(2);

  signalA.addDependency(signalB);
  ```

#### `removeDependency(dependency)`

- **Description:** Remove a dependency from the signal.

- **Parameters:**
  - `dependency`: The `Alpena` instance to remove.

- **Usage:**

  ```javascript
  signalA.removeDependency(signalB);
  ```

#### `removeAllDependencies()`

- **Description:** Remove all dependencies from the signal.

- **Usage:**

  ```javascript
  signal.removeAllDependencies();
  ```

#### `update(updateFunction)`

- **Description:** Update the signal's value using a function that receives the current value and returns a new value.

- **Parameters:**
  - `updateFunction(currentValue)`: A function that takes the current value and returns a new value.

- **Throws:**
  - Error if the returned value is undefined or of a different type than the current value.

- **Usage:**

  ```javascript
  signal.update((current) => current + 1);
  ```

#### `get()`

- **Description:** Get the current value of the signal.

- **Returns:** The current value.

- **Usage:**

  ```javascript
  const value = signal.get();
  ```

#### `set(value)`

- **Description:** Set a new value to the signal. Notifies subscribers if the value has changed.

- **Parameters:**
  - `value`: The new value to set.

- **Usage:**

  ```javascript
  signal.set(42);
  ```

## Examples

### Basic Usage

```javascript
import Alpena from './Alpena.js';

// Create a new signal
const counter = new Alpena(0);

// Subscribe to the signal
const unsubscribe = counter.subscribe((value) => {
  console.log(`Counter: ${value}`);
});

// Update the signal's value
counter.set(1); // Output: Counter: 1
counter.update((prev) => prev + 1); // Output: Counter: 2

// Unsubscribe when no longer needed
unsubscribe();
```

### Dependency Management

```javascript
import Alpena from './Alpena.js';

// Create signals
const firstName = new Alpena('Alice');
const lastName = new Alpena('Smith');
const fullName = new Alpena('Ms.');

// Define dependencies
fullName.addDependency(firstName, lastName);

// Subscribe to fullName changes
fullName.subscribe((value, first, last) => {
  const newFullName = `${value} ${first} ${last}`;
  console.log(`Full Name: ${newFullName}`);
});

// Change dependencies
firstName.set('Jane'); // Output: Full Name: Ms. Jane Smith
lastName.set('Doe'); // Output: Full Name: Ms. Jane Doe
```

### Working with Arrays and Objects

```javascript
import Alpena from './Alpena.js';

// Signal with an array
const items = new Alpena(['apple', 'banana']);

// Subscribe to changes
items.subscribe((list) => {
  console.log('Items:', list.join(', '));
});

// Update the array immutably
items.update((prevItems) => [...prevItems, 'cherry']); // Output: Items: apple, banana, cherry

// Signal with an object
const user = new Alpena({ name: 'Alice', age: 30 });

user.subscribe((data) => {
  console.log(`User: ${data.name}, Age: ${data.age}`);
});

// Update the object immutably
user.update((prevUser) => ({ ...prevUser, age: prevUser.age + 1 })); // Output: User: Alice, Age: 31
```

## Utilities and Helpers

Alpena includes utility functions for internal use, such as deep equality checks (`deepEqual`), type checking (`getType`), and determining if a value is primitive (`isPrimitive`). These ensure signals behave predictably when dealing with complex data types.

## Best Practices

- **Avoid Direct Mutation:** When working with objects or arrays, use immutable update patterns to ensure changes are detected. For example, use spread operators or methods that return new arrays or objects.

  ```javascript
  // Good
  signal.update((data) => ({ ...data, newProp: value }));

  // Bad (direct mutation)
  signal.value.newProp = value;
  ```

- **Consistent Types:** Ensure that updates to a signal's value maintain the same data type to prevent errors.

- **Unsubscribe When Necessary:** To prevent memory leaks, unsubscribe from signals when they are no longer needed, especially in frameworks or environments where components might unmount or be destroyed.

- **Manage Dependencies Carefully:** When adding dependencies, remember that any change in a dependency will trigger notifications. Only add dependencies that are essential for the signal's logic.

- **Handle Errors Gracefully:** Subscriber callbacks should handle potential errors to prevent unintended side effects.

  ```javascript
  signal.subscribe((value) => {
    try {
      // Handle value
    } catch (error) {
      console.error('Error handling signal value:', error);
    }
  });
  ```
