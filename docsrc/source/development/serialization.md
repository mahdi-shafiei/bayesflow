# Serialization: Enable Model Saving & Loading

Serialization deals with the problem of storing objects to disk, and loading them at a later point in time.
This is straight-forward for data structures like numpy arrays, but for classes with custom behavior it is somewhat more complex.

Please refer to the Keras guide [Save, serialize, and export models](https://keras.io/guides/serialization_and_saving/) for an introduction, and [Customizing Saving and Serialization](https://keras.io/guides/customizing_saving_and_serialization/) for advanced concepts.

The basic idea is: by storing the arguments of the constructor of a class (i.e., the arguments of the `__init__` function), we can later construct an object similar to the one we have stored, except for the weights and other stateful content.
As the structure is identical, we can then map the stored weights to the newly constructed object.
The caveat is that all arguments have to be either basic Python objects (like int, float, string, bool, ...) or themselves serializable.
If they are not, we have to manually specify how to serialize them, and how to load them later on.
One important example is that types are not serializable.
As we want/need to pass them in some places, we have to resort to some custom behavior, that is described below.

## Serialization Utilities

BayesFlows serialization utilities can be found in the {py:mod}`~bayesflow.utils.serialization` module.
We mainly provide three convenience functions:

- The {py:func}`~bayesflow.utils.serialization.serializable` decorator wraps the `keras.saving.register_keras_serializable` function to ensure consistent naming of the `package` argument within the library.
- The {py:func}`~bayesflow.utils.serialization.serialize` function, which adds support for serializing classes.
- Its counterpart {py:func}`~bayesflow.utils.serialization.deserialize`, adds support to deserialize classes.

## Usage

To use the adapted serialization functions, you have to use them in the `get_config` and `from_config` method. Please refer to existing classes in the library for usage examples.

### The `serializable` Decorator

To make serialization as little confusing as possible, as well as providing stability even when moving classes around, we provide the `package` argument explicitly for each class.
The naming should respect the following naming scheme: Take the module the class resides in (for example, `bayesflow.adapters.transforms.standardize`), and truncate the path to depth two (`bayesflow.adapters`).
In cases where this convention cannot be followed, set `disable_module_check` to `True`, and describe why a different name was necessary.
Changing `package` breaks backwards-compatibility for serialization, so it should be avoided whenever possible.
If you move a class to a different module (without changing the class itself), keep the `package` and set `disable_module_check` to `True`.
This may later be adapted in a release that breaks backward compatiblity anyways.
