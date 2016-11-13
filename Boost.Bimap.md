# Глава 13. Boost.Bimap

Библиотека [Boost.Bimap](http://www.boost.org/libs/bimap) основана на Boost.MultiIndex, и обеспечивает контейнер, который может быть использован будучи не определенным сначала. Этот контейнер подобен `std::map`, но поддерживает просмотр значений с обеих сторон. Boost.Bimap позволяет создавать карты, где любая сторона может быть ключом, в зависимости от того, как вы получаете доступ к карте. Когда Вы получаете доступ к левой стороне как к ключу, правая сторона - значение, и наоборот.

<a name="example131"></a>
### Пример 13.1. Использование `boost::bimap`
```c++
#include <boost/bimap.hpp>
#include <string>
#include <iostream>

int main()
{
  typedef boost::bimap<std::string, int> bimap;
  bimap animals;

  animals.insert({"cat", 4});
  animals.insert({"shark", 0});
  animals.insert({"spider", 8});

  std::cout << animals.left.count("cat") << '\n';
  std::cout << animals.right.count(8) << '\n';
}
```

`boost::bimap` определен в `boost/bimap.hpp`, и поддерживает две переменные экземпляра, **left** и **right**, которые могут использоваться, чтобы получить доступ к двум контейнерам типа `std::map` которые объединены `boost::bimap`. В [Примере 13.1](#example131), **left** использует ключи типа `std::string` для доступа к контейнеру, и **right** использует ключи типа `int`.

Помимо поддержки доступа к отдельным записям используя левый или правый контейнер, `boost::bimap` позволяет посмотреть записи как связи (Смотри [Пример 13.2](#example132)).


<a name="example132"></a>
### Пример 13.2. Доступ к связям
```c++
#include <boost/bimap.hpp>
#include <string>
#include <iostream>

int main()
{
  typedef boost::bimap<std::string, int> bimap;
  bimap animals;

  animals.insert({"cat", 4});
  animals.insert({"shark", 0});
  animals.insert({"spider", 8});

  for (auto it = animals.begin(); it != animals.end(); ++it)
    std::cout << it->left << " has " << it->right << " legs\n";
}
```

Не обязательно использовать **left** или **right** для доступа к записям. Перебирая записи, левая и правая части индивидуальной записи, становятся доступны через итератор.

В то время как `std::map` сопровождается контейнером под названием `std::multimap`, который может хранить несколько записей, используя один и тот же ключ, у `boost::bimap` нет такого эквивалента. Однако это не означает, что хранение нескольких записей с тем же самым ключом внутри контейнера `boost::bimap` невозможно. Строго говоря, два требуемых параметра шаблона указывают контейнеру типы **left** и **right**, но не типы элементов для хранения. Если тип контейнера не определен, тип `boost::bimaps::set_of` будет использоваться по умолчанию. Этот контейнер, как `std::map`, принимает записи только с уникальным ключом.

<a name="example133"></a>
### Пример 13.3. Использование `boost::bimaps::set_of` в явном виде
```c++
#include <boost/bimap.hpp>
#include <string>
#include <iostream>

int main()
{
  typedef boost::bimap<boost::bimaps::set_of<std::string>,
    boost::bimaps::set_of<int>> bimap;
  bimap animals;

  animals.insert({"cat", 4});
  animals.insert({"shark", 0});
  animals.insert({"spider", 8});

  std::cout << animals.left.count("spider") << '\n';
  std::cout << animals.right.count(8) << '\n';
}
```
[Пример 13.3](#example133) определяет `boost::bimaps::set_of`. Другие типы контейнеров, кроме `boost::bimaps::set_of` могут быть использованы для настройки `boost::bimap`.

<a name="example134"></a>
### Пример 13.4. Предоставление дубликатов с `boost::bimaps::multiset_of`
```c++
#include <boost/bimap.hpp>
#include <boost/bimap/multiset_of.hpp>
#include <string>
#include <iostream>

int main()
{
  typedef boost::bimap<boost::bimaps::set_of<std::string>,
    boost::bimaps::multiset_of<int>> bimap;
  bimap animals;

  animals.insert({"cat", 4});
  animals.insert({"shark", 0});
  animals.insert({"dog", 4});

  std::cout << animals.left.count("dog") << '\n';
  std::cout << animals.right.count(4) << '\n';
}
```
[Пример 13.4](#example134) использует контейнер типа `boost::bimaps::multiset_of`, который определен в `boost/bimap/multiset_of.hpp`. Он работает как `boost::bimaps::set_of`, за исключением того, что ключи не должны быть уникальными. [Пример 13.4](#example134) успешно покажет **2** ища животных с четырьмя ногами.

Поскольку `boost::bimaps::set_of` используется по умолчанию для контейнеров типа `boost::bimap`, заголовочный файл `boost/bimap/set_of.hpp` не должен включаться явно. Однако, используя другие контейнерные типы, соответствующие им заголовочные файлы должны быть включены.

В дополнение к классам, показанным выше, Boost.Bimap обеспечивает следующее: `boost::bimaps::unordered_set_of`, `boost::bimaps::unordered_multiset_of`, `boost::bimaps::list_of`, `boost::bimaps::vector_of`, и `boost::bimaps::unconstrained_set_of`. За исключением `boost::bimaps::unconstrained_set_of`, все другие контейнерные типы работают точно так же, как их "коллеги" из стандартной библиотеки.

<a name="example135"></a>
### Пример 13.5. Отключение одной стороны с `boost::bimaps::unconstrained_set_of`
```c++
#include <boost/bimap.hpp>
#include <boost/bimap/unconstrained_set_of.hpp>
#include <boost/bimap/support/lambda.hpp>
#include <string>
#include <iostream>

int main()
{
  typedef boost::bimap<std::string,
    boost::bimaps::unconstrained_set_of<int>> bimap;
  bimap animals;

  animals.insert({"cat", 4});
  animals.insert({"shark", 0});
  animals.insert({"spider", 8});

  auto it = animals.left.find("cat");
  animals.left.modify_key(it, boost::bimaps::_key = "dog");

  std::cout << it->first << '\n';
}
```
`boost::bimaps::unconstrained_set_of` может использоваться, чтобы исключить одну сторону boost::bimap. В [Примере 13.5](#example135), `boost::bimap` ведет себя как `std::map`. Вы не можете получить доступ к **right**, чтобы искать животных по ногам.

[Пример 13.5](#example135) иллюстрирует другую причину, почему может иметь смысл предпочитать `boost::bimap` нежели `std::map`. Так как Boost.Bimap основан на Boost.MultiIndex, доступны функции-члены из Boost.MultiIndex. [Пример 13.5](#example135) изменяет ключ, используя modify_key() – то, что невозможно с `std::map`.

Обратите внимание, как изменяется ключ. Новое значение присваивается текущему ключу, с использованием **boost::bimaps::_key**, который определен в `boost/bimap/support/lambda.hpp`.

`boost/bimap/support/lambda.hpp` также определяет **boost::bimaps::_data**. При вызове функции-члена `modify_data()`, **boost::bimaps::_data** может использоваться, чтобы изменить значение в контейнере типа `boost::bimap`.
