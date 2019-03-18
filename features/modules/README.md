⬅ [Home](/mamba_doc/README.md)

⬅ [Features](/mamba_doc/features/README.md)

# Module Features

## Contents

### [Script](/mamba_doc/features/modules/script.md)
### [Types](/mamba_doc/features/modules/types.md)
### [State and Stateless](/mamba_doc/features/modules/state_stateless.md)

## Introduction

A module is one of the following:

Module   | Description
---------|--------------
Script   | A sequence of Instructions.
Type     | A blueprint for a class, defines the behaviour of a class.
Statless or stateful    | A blueprint for an instance of an object. Defines the behaviour of that object, and may implement a type.

A file may contain either a script accompanied by functions, or a mix of types, utils and classes. 
Furthermore, it is generally best to have at most one type per file for readability sake.
