[![Build Status](https://travis-ci.com/mikeloomisgg/cppack.svg?branch=master)](https://travis-ci.com/mikeloomisgg/cppack)
# cppack
A modern (c++17 required) implementation of the [msgpack spec](https://github.com/msgpack/msgpack/blob/master/spec.md).

Msgpack is a binary serialization specification. It allows you to save and load application objects like classes and structs over networks, to files, and between programs and even different languages.

Check out [this blog](https://mikeloomisgg.github.io/2019-07-02-making-a-serialization-library/) for my rational creating this library.

## Features
- Fast and compact
- Full test coverage
- Easy to use
- Automatic type handling
- Open source MIT license
- Easy error handling

### Single Header only template library
Want to use this library? Just #include the header and you're good to go. Its less than 1000 lines of code.


### Cereal style packaging
Easily pack objects into byte arrays using a pack free function:

```c++
struct Person {
  std::string name;
  uint16_t age;
  std::vector<std::string> aliases;

  template<class T>
  void pack(T &pack) {
    pack(name, age, aliases);
  }
};

int main() {
    auto person = Person{"John", 22, {"Ripper", "Silverhand"}};

    auto data = msgpack::pack(person); // Pack your object
    auto john = msgpack::unpack<Person>(data); // Unpack it
}
```

### Serialize to and from JSON

With the help of the [json](https://github.com/nlohmann/json.git) library and the new functions `as_array()`, `as_map()`, `map()`, `array()`, and `item()` it is now possible to conveniently serialize via msgpack format to and from JSON. Below is a complete example demonstrating possible usage.

```c++
#include <iostream>

#include <nlohmann/json.hpp>
#include <msgpack/msgpack.hpp>

using json = nlohmann::json;

#define TO_STRING(name)   #name
#define ITEM(name)        item(TO_STRING(name), name)

struct Color {
    float red, green, blue;

    template<class T>
    void pack(T& pack) {
        pack.as_array(red, green, blue);
    }
};

struct Model {
    std::string name;
    std::string brand;
    int year;
};

struct Car {
    Model model;
    Color color;

    template<class T>
    void pack(T& pack) {
        pack.as_map(
            pack.item("model", pack.map(
                pack.item("name", model.name),
                pack.item("brand", model.brand),
                pack.item("year", model.year)
            )),
            pack.ITEM(color)
        );
    }
};

struct Person {
    std::string first_name;
    std::string last_name;
    std::vector<Car> cars;

    template<class T>
    void pack(T& pack) {
        pack(pack.map(pack.ITEM(first_name), pack.ITEM(last_name), pack.ITEM(cars)));
    }
};

int main(int argc, char* argv[])
{
    Person person { .first_name = "John", .last_name = "Doe", .cars = {
        { { "Porsche 911", "Porsche", 2017 }, { 0.9f, 0.8f, 0.7f } },
        { { "BMW X5", "BMW", 2019 }, { 0.5f, 0.9f, 0.8f } }
    } };

    auto data = msgpack::pack(person);

    std::cout << std::setw(2) << json::from_msgpack(data) << std::endl;

    auto unpacked_person = msgpack::unpack<Person>(data);

    std::cout << "unpacked and re-packed:" << std::endl;

    std::cout << std::setw(2) << json::from_msgpack(msgpack::pack(unpacked_person)) << std::endl;

    return 0;
}
```

JSON output:

```json
{
  "cars": [
    {
      "color": [
        0.8999999761581421,
        0.800000011920929,
        0.699999988079071
      ],
      "model": {
        "brand": "Porsche",
        "name": "Porsche 911",
        "year": 2017
      }
    },
    {
      "color": [
        0.5,
        0.8999999761581421,
        0.800000011920929
      ],
      "model": {
        "brand": "BMW",
        "name": "BMW X5",
        "year": 2019
      }
    }
  ],
  "first_name": "John",
  "last_name": "Doe"
}
```

[More Examples](msgpack/tests/examples.cpp)


### Roadmap
- Support for extension types
  - The msgpack spec allows for additional types to be enumerated as Extensions. If reasonable use cases come about for this feature then it may be added.
- Name/value pairs
  - The msgpack spec uses the 'map' type differently than this library. This library implements maps in which key/value pairs must all have the same value types.
- Endian conversion shortcuts
  - On platforms that already hold types in big endian, the serialization could be optimized using type traits.
