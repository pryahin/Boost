# Глава 12. Boost.MultiIndex

[Boost.MultiIndex](http://www.boost.org/libs/multi_index ) позволяет определить контейнеры, которые поддерживают произвольное число интерфейсов. В то время как `std::vector` обеспечивает интерфейс, который поддерживает прямой доступ к элементам с индексом, а `std::set` предоставляет интерфейс, который сортирует элементы, Boost.MultiIndex позволяет определить контейнеры, которые поддерживают оба интерфейса. Такой контейнер может быть использован для доступа к элементам с использованием индекса и в отсортированной манере.

Boost.MultiIndex может быть использован, если элементы должны быть доступны по-разному, и, как правило, должны храниться в нескольких контейнерах. Вместо того, чтобы хранить элементы как вектор и набор, а затем непрерывно синхронизировать контейнеры, вы можете определить контейнер с Boost.MultiIndex, который обеспечивает  интерфейс вектора и набора.

Boost.MultiIndex также имеет смысл, если вам нужно именть доступ к элементам на основе множества различных свойств. В [примере 12.1](#example121), животные просматриваются по имени и по количеству лап. Без Boost.MultiIndex потербуется два хэш-контейнера - один для поиска животных по имени, а друго, чтобы искать их по количеству лап.

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

При использовании Boost.MultiIndex, первый шаг заключается в определении нового контейнера. Вы должны решить, какие интерфейсы ваш новый контейнер должен поддерживать и к каким свойствам элемента он должен получить доступ.

Класс `boost::multi_index::multi_index_container`, который определен в `boost/multi_index_container.hpp`, используется для каждого определения контейнера. Это шаблон класса, который требует по крайней мере двух параметров. Первый параметр это тип элементов, которые должен хранить контейнер - в [Примере 12.1](#example121) это определенный пользователем класс названный `animal`. Второй параметр используется, чтобы обозначить различные индексы, которые должен обеспечить контейнер.

Главное преимущество контейнеров, основанных на Boost.MultiIndex - то, что Вы можете получить доступ к элементам через различные интерфейсы. При определении нового контейнера, вы можете указать количество и тип интерфейсов. Контейнер в [Примере 12.1](#example121) нужен для поддержки поиска животных по имени или количеству лап, так что определено два интерфейса. Boost.MultiIndex называет эти интерфейсы индексами - именно отсюда происходит название библиотеки.

Интерфейсы определяются с помощью класса `boost::multi_index::indexed_by`. Каждый интерфейс передается как параметр шаблона. Два интерфейса типа `boost::multi_index::hashed_non_unique`, которые определены в `boost/multi_index/hashed_index.hpp`, используются в [Примере 12.1](#example121). Используюя эти интерфесы контейнер ведет себя как `std::unordered_set`, и ищет значения используя хэш-значение.