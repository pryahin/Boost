# Глава 12. Boost.MultiIndex

[Boost.MultiIndex](http://www.boost.org/libs/multi_index) позволяет определить контейнеры, которые поддерживают произвольное число интерфейсов. В то время как `std::vector` обеспечивает интерфейс, который поддерживает прямой доступ к элементам с индексом, а `std::set` предоставляет интерфейс, который сортирует элементы, Boost.MultiIndex позволяет определить контейнеры, которые поддерживают оба интерфейса. Такой контейнер может быть использован для доступа к элементам с использованием индекса и в отсортированной манере.

Boost.MultiIndex может быть использован, если элементы должны быть доступны по-разному, и, как правило, должны храниться в нескольких контейнерах. Вместо того, чтобы хранить элементы как вектор и множество, а затем непрерывно синхронизировать контейнеры, вы можете определить контейнер с Boost.MultiIndex, который обеспечивает интерфейс вектора и множества.

Boost.MultiIndex также имеет смысл использовать, если вам нужно иметь доступ к элементам на основе множества различных свойств. В [примере 12.1](#example121), животные просматриваются по имени и по количеству лап. Без Boost.MultiIndex потребуется два хэш-контейнера - один для поиска животных по имени, а другой, чтобы искать их по количеству лап.

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

Интерфейсы определяются с помощью класса `boost::multi_index::indexed_by`. Каждый интерфейс передается как параметр шаблона. Два интерфейса типа `boost::multi_index::hashed_non_unique`, которые определены в `boost/multi_index/hashed_index.hpp`, используются в [Примере 12.1](#example121). Используя эти интерфейсы, контейнер ведет себя как `std::unordered_set`, и ищет значения используя хэш-значение.

Класс `boost::multi_index::hashed_non_unique` это шаблон, и ожидает в качестве единственного параметра класс, который вычисляет хэш-значения. Поскольку оба интерфейса контейнера должны рассматривать животных, один интерфейс вычисляет хэш-значения по имени, в то время как другой интерфейс вычисляет по числу лап.

Boost.MultiIndex предлагает вспомогательный шаблон класса `boost::multi_index::member`, который определен в `boost/multi_index/member.hpp`, чтобы получить доступ к переменной экземпляра. Как видно из [Примера 12.1](#example121), некоторые параметры были заданы, чтобы позволить `boost::multi_index::member` знать, какая переменная экземпляра `animal` должна быть доступна и какой тип переменная экземпляра имеет.

Даже при том, что определение `animal_multi` на первый взгляд выглядит сложным, класс работает как карта(map). Имя и количество лап животного можно рассматривать как пару ключ/значение. Преимущество контейнера `animal_multi` над картой, такой как `std::unordered_map` в том, что животные могут рассматриваться по имени или по количеству лап. `animal_multi` поддерживает два интерфейса, один на основе имени и один на основе числа лап. Интерфейс определяет, какая переменная экземпляра является ключом, а какая переменная экземпляра является значением.

Чтобы получить доступ к контейнеру MultiIndex, вам необходимо выбрать интерфейс. Если вы непосредственно получаете доступ к объекту **animals** с помощью `insert()` или `count()`, используется первый интерфейс. В [Примере 12.1](#example121), это хэш-контейнер для имени переменной экземпляра. Если вам нужен другой интерфейс, вы должны выбрать его явно.

Интерфейсы нумеруются, начиная с индекса 0. Чтобы получить доступ ко второму интерфейсу - как показано в [Примере 12.1](#example121) - нужно вызвать функцию `get()` и передать индекс нужного интерфейса в качестве параметра шаблона.

Возвращаемое значение `get()` выглядит достаточно сложным. Он обращается к классу контейнера MultiIndex под названием `nth_index`, который, опять же, является шаблоном. Индекс интерфейса, который будет использоваться должен быть указан в качестве параметра шаблона. Этот индекс должен быть таким же, как переданный `get()`. Последним шагом является получение доступа к определению типа `type` из `nth_index`. Значение `type` представляет собой тип соответствующего интерфейса. Следующие примеры используют ключевое слово `auto` для упрощения кода.

Несмотря на то, что вам не нужно знать специфику интерфейса, так как они автоматически выводятся из `nth_index` и `type`, вы все равно должны понять, какой интерфейс доступен. Поскольку интерфейсы нумеруются последовательно в определении контейнера, это можно узнать легко, так как индекс передается как `get()` и `nth_index`. Таким образом, `legs_index` - хэш-интерфейс, который смотрит животных по лапам.

Поскольку данные, такие как имена и лапы могут быть ключами контейнера MultiIndex, они не могут быть произвольно изменены. Если число лап изменяется после того, как животное было найдено по имени, интерфейс использующий лапы в качестве ключа, не будет знать об изменениях и не будет знать, что новое значение хеш-функции должно быть изменено.

Подобно тому, как ключи в контейнере типа `std::unordered_map` не могут быть изменены, не могут быть изменены данные хранящиеся в контейнере MultiIndex. Строго говоря, все данные, хранящиеся в контейнере MultiIndex постоянны. Это включает в себя переменные экземпляров, которые не используются никакими интерфейсами. Даже если ни один интерфейс не получает доступ к **legs(лапам)**, **legs(лапы)** не могут быть изменены.

Для того, чтобы избежать необходимости удаления элементов из контейнера MultiIndex и вставить новые, Boost.MultiIndex предоставляет функции для непосредственного изменения значений. Поскольку эти функции работают с самим MultiIndex контейнером, и потому что ни один элемент в контейнере не изменяется непосредственно, все интерфейсы будут уведомлены и смогут вычислить новые хеш-значения.

<a name="example121"></a>
### Пример 12.2. Изменение элементов в MultiIndex контейнере с `modify()`
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

  auto &legs_index = animals.get<1>();
  auto it = legs_index.find(4);
  legs_index.modify(it, [](animal &a){ a.name = "dog"; });

  std::cout << animals.count("dog") << '\n';
}
```
Каждый предлагаемый Boost.MultiIndex интерфейс предоставляет функцию `modify()`, которая работает непосредственно с контейнером. Объект который должен быть изменен, определен через итератор, переданного в качестве первого параметра для `modify()`. Второй параметр является функцией или функциональным объектом, который ожидает, в качестве единственного параметра объект типа, хранящегося в контейнере. Функция или объект функции могут изменять элемент столько, сколько хотят. [Пример 12.2](#example121) показывает, как использовать функцию `modify()`, чтобы изменить элемент.

До сих пор только один интерфейс был введен: `boost::multi_index::hashed_non_unique`, который вычисляет хэш-значение, которое не должно быть уникальным. Для того, чтобы гарантировать, что ни одно значение не сохраняется два раза, используется `boost::multi_index::hashed_unique`. Обратите внимание, что значения не могут быть сохранены, если они не удовлетворяют требованиям всех интерфейсов конкретного контейнера. Если один интерфейс не позволяет хранить значения несколько раз, то не имеет значения, позволяет другой интерфейс или нет.

<a name="example123"></a>
### Пример 12.3 Контейнер MultiIndex с `boost::multi_index::hashed_unique`
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
    hashed_unique<
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
  animals.insert({"dog", 4});

  auto &legs_index = animals.get<1>();
  std::cout << legs_index.count(4) << '\n';
}
```

Контейнер в [Примере 12.3](#example123) использует `boost::multi_index::hashed_unique` в качестве второго интерфейса. Это означает, что никакие два животных с одинаковым числом лап не могут быть сохранены в контейнере, так как хэш-значения будут одинаковыми.

Пример пытается хранить собаку, которая имеет то же количество лап, как уже у сохраненной кошки. Поскольку это нарушает требование наличия уникальных хэш-значений для второго интерфейса, собака не будет сохранена в контейнере. Поэтому при поиске животных с четырьмя лапами, программа выводит на экран 1, потому что только кот был сохранен и посчитан.

<a name="example124"></a>
### Пример 12.4. Интерфейсы `sequenced`, `ordered_non_unique` и `random_access`
```c++
#include <boost/multi_index_container.hpp>
#include <boost/multi_index/sequenced_index.hpp>
#include <boost/multi_index/ordered_index.hpp>
#include <boost/multi_index/random_access_index.hpp>
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
    sequenced<>,
    ordered_non_unique<
      member<
        animal, int, &animal::legs
      >
    >,
    random_access<>
  >
> animal_multi;

int main()
{
  animal_multi animals;

  animals.push_back({"cat", 4});
  animals.push_back({"shark", 0});
  animals.push_back({"spider", 8});

  auto &legs_index = animals.get<1>();
  auto it = legs_index.lower_bound(4);
  auto end = legs_index.upper_bound(8);
  for (; it != end; ++it)
    std::cout << it->name << '\n';

  const auto &rand_index = animals.get<2>();
  std::cout << rand_index[0].name << '\n';
}
```

[Пример 12.4](#example124) представляет три последних интерфейса Boost.MultiIndex: `boost::multi_index::sequenced`, `boost::multi_index::ordered_non_unique`, и `boost::multi_index::random_access`.

Интерфейс `boost::multi_index::sequenced` позволяет рассматривать MultiIndex контейнер как список типа `std::list`. Элементы хранятся в указанном порядке.

С помощью интерфейса `boost::multi_index::ordered_non_unique`, объекты автоматически сортируются. Этот интерфейс требует, чтобы вы при определении контейнера указали критерий сортировки. [Пример 12.4](#example124) сортирует объекты типа `animal` по количеству лап с помощью вспомогательного класса `boost::multi_index::member`.

`boost::multi_index::ordered_non_unique` предоставляет специальные функции, чтобы найти определенные диапазоны в пределах отсортированных значений. Используя `lower_bound()` и `upper_bound()`, программа ищет животных, которые имеют по крайней мере четыре лапы, но не больше, чем восемь лап. Поскольку они требуют, чтобы элементы были сортированы, эти функции не предоставляются другими интерфейсами.

Окончательный введенный интерфейс `boost::multi_index::random_access`, позволяет рассматривать MultiIndex контейнер как вектор типа `std::vector`. Двумя наиболее известными функциями являются `operator[]` и `at()`.

`boost::multi_index::random_access` включает в себя `boost::multi_index::sequenced`. С `boost::multi_index::random_access`, все функции `boost::multi_index::sequenced` также доступны.

Теперь, когда мы рассмотрели четыре интерфейса `Boost.MultiIndex`, остаток главы сосредотачивается на *ключевых экстракторах*. Один из ключевых экстракторов уже был введен: `boost::multi_index::member`, который определен в `boost/multi_index/member.hpp`. Этот вспомогательный класс называется ключевым экстрактором, потому что он позволяет определить, какая переменная экземпляра класса должна использоваться в качестве ключа интерфейса.

[Пример 12.5](#example125) вводит еще два ключевых экстрактора.

<a name="example125"></a>
### Пример 12.5. Ключевые экстракторы `identity` и `const_mem_fun`
```c++
#include <boost/multi_index_container.hpp>
#include <boost/multi_index/ordered_index.hpp>
#include <boost/multi_index/hashed_index.hpp>
#include <boost/multi_index/identity.hpp>
#include <boost/multi_index/mem_fun.hpp>
#include <string>
#include <utility>
#include <iostream>

using namespace boost::multi_index;

class animal
{
public:
  animal(std::string name, int legs) : name_{std::move(name)},
    legs_(legs) {}
  bool operator<(const animal &a) const { return legs_ < a.legs_; }
  const std::string &name() const { return name_; }
private:
  std::string name_;
  int legs_;
};

typedef multi_index_container<
  animal,
  indexed_by<
    ordered_unique<
      identity<animal>
    >,
    hashed_unique<
      const_mem_fun<
        animal, const std::string&, &animal::name
      >
    >
  >
> animal_multi;

int main()
{
  animal_multi animals;

  animals.emplace("cat", 4);
  animals.emplace("shark", 0);
  animals.emplace("spider", 8);

  std::cout << animals.begin()->name() << '\n';

  const auto &name_index = animals.get<1>();
  std::cout << name_index.count("shark") << '\n';
}
```

Ключевой экстрактор `boost::multi_index::identity`, определенный в `boost/multi_index/identity.hpp`, использует элементы, хранящиеся в контейнере в виде ключей. Это требует от класса `animal`, чтобы он был сортируемым, так как объекты типа `animal `будут использоваться в качестве ключа для интерфейса `boost::multi_index::ordered_unique`. В [Примере 12.5](#example125), это достигается за счет перегруженного `operator<`.

Файл заголовка `boost/multi_index/mem_fun.hpp` определяет два ключевых экстрактора - `boost::multi_index::const_mem_fun` и `boost::multi_index::mem_fun` - используют возвращаемое значение функции в качестве ключа. В [Примере 12.5](#example125), таким образом используется возвращаемое значение `name()`. `boost::multi_index::const_mem_fun` используется для константных функций, в то время как `boost::multi_index::mem_fun` используется для неконстантных функций.

Boost.MultiIndex предлагает еще два ключевых экстрактора: `boost::multi_index::global_fun` и `boost::multi_index::composite_key`. Первый может быть использован для свободно стоящих или статических функций, а второй позволяет проектировать ключевой экстрактор, составленный из нескольких других ключевых экстракторов.
