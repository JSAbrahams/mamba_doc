# Safety Features

## Contents

### [Types](/features/safety/00_types.md)

### [Error Handling](/features/safety/01_error_handling.md)

### [Null Safety](/features/safety/02_null_safety.md)

### [Generics](/features/safety/03_generics.md)

## Introduction

There are several safety features baked into the language to make it easier to catch bugs early on.

We have a type system so we can define functions which may only manipulate a certain type of object.
We also have type refinement features to further enhance function definitions.
We can also specify what type of object they return.
To extend this we have also added generics to the language, which can be useful when for instance defining collections of objects.

We have certain error handling mechanisms built in, which encourage error handling on site help to make it clear visually where errors might occur.
To avoid undefined behaviour, we also baked null safety features into the language.
