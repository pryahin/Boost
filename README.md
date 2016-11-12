# Глава 12. Boost.MultiIndex

[Boost.MultiIndex](http://www.boost.org/libs/multi_index ) позволяет определить контейнеры, которые поддерживают произвольное число интерфейсов. В то время как `std::vector` обеспечивает интерфейс, который поддерживает прямой доступ к элементам с индексом, а `std::set` предоставляет интерфейс, который сортирует элементы, Boost.MultiIndex позволяет определить контейнеры, которые поддерживают оба интерфейса. Такой контейнер может быть использован для доступа к элементам с использованием индекса и в отсортированной манере.

Boost.MultiIndex может быть использован, если элементы должны быть доступны по-разному, и, как правило, должны храниться в нескольких контейнерах. Вместо того, чтобы хранить элементы как вектор и набор, а затем непрерывно синхронизировать контейнеры, вы можете определить контейнер с Boost.MultiIndex, который обеспечивает  интерфейс вектора и набора.


Boost.MultiIndex также имеет смысл, если вам нужно именть доступ к элементам на основе множества различных свойств. В [примере 12.1](#example121), животные просматриваются по имени и по количеству ног. Без Boost.MultiIndex потербуется два хэш-контейнера - один для поиска животных по имени, а друго, чтобы искать их по количеству ног.

<a name="example121"></a>
### Пример 12.1. Использование `boost::multi_index::multi_index_container`
```c++
#include <boost/multi_index_container.hpp>
#include <boost/multi_index/hashed_index.hpp>
#include <boost/multi_index/member.hpp>
#include <string>
#include <iostream>

using namespace boost::multi_index;

struct animal
{
  std::string name;
  int legs;
};

typedef multi_index_container<
  animal,
  indexed_by<
    hashed_non_unique<
      member<
        animal, std::string, &animal::name
      >
    >,
    hashed_non_unique<
      member<
        animal, int, &animal::legs
      >
    >
  >
> animal_multi;

int main()
{
  animal_multi animals;

  animals.insert({"cat", 4});
  animals.insert({"shark", 0});
  animals.insert({"spider", 8});

  std::cout << animals.count("cat") << '\n';

  const animal_multi::nth_index<1>::type &legs_index = animals.get<1>();
  std::cout << legs_index.count(8) << '\n';
}
```
