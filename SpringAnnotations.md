Спринг это фреймворк который дает нам инверсию контроля

- Конструктор вызывается раньше чем внедряется зависимость в поле
- Внедрение зависимости через аннотацию AutowiredAnnotationBeanPostProcessor
- XML Конфигурация имеет наивысший приоритет (перезапишет все что сделано после аннотаций)
- Методы и свойства могут быть приватными (тут пахнет рефлексией) (может внедрить зависимость)

Почему xml актуален?
- инкапсулированный файл
- Направленная конфигурация
- Проще изменить файлик чем перекомпилировать Java Config


Стадии понимания сингльтона
1. Студент
- синглтон это виски :D
2. Стажер
```JAVA
public class MySingleton {
  private static MySingleton singleton;

  private MySingleton() {}  // приватный конструктор

  // Возращаем экземпляр через getInstance
  public static MySingleton getInstance() {
    if (singleton == null) {
      singleton = new MySingleton();
    }
    return singleton;
  }
}
```
- Проблемы с многопоточностью (получим какой то даблтон или триплтон)
3. Junior Software Engineer
```
public class MySingleton {
  private static MySingleton singleton;
  private MySingleton() {}

  // Make it synchronized!
  public synchronized static MySingleton getInstance() {
    if (singleton == null) {
      singleton = new MySingleton();
    }
    return singleton;
  }
}
```
- А Перформанс?
4. Senior Software Engineer
- лочим не метод, а только создание самого инстанса
- Out of order exceution - оптимизация чтобы безконца не класть в хип, а вычислить в стеке и только потом вставить (тут уже второй if безсмысленный - оптимизация его нафиг выкинет)
- Два основных вида памяти (Стэк и Хип)
- Хип может только читать и записывать
```Java
  // Make INIT synchronized!
  public  static MySingleton getInstance() {
    if (singleton == null) {  // первый чек чтобы зайти в синхронизированный блок
      synchronized (MySingleton.class) {
        // нужен дабл чек чтобы понять есть ли другой залоченный поток, который хочет создать сингльтон
        if(singleton == null) {
          singleton = new MySingleton();
        }
      }
    }
    return singleton;
  }
```
5. Lead Software Engineer
```
// use volatile
private static volatile MySingleton singleton;  // не нужно включать оптимизацию, но будет работать медленней (Но все же есть баги в JVM)
  private MySingleton() {}

  public  static MySingleton getInstance() {
    if (singleton == null) { 
      synchronized (MySingleton.class) {
        if(singleton == null) {
          singleton = new MySingleton();
        }
      }
    }
    return singleton;
  }
```
- eager Singleton - инициализируется когда загружается класс сиингльтона
- Сингльтон через enum
```Java
public enum SingletonEnum {
  INSTANCE;
  public void doWork() {
    System.out.println("INSTACNCE IS working");
  }

  public static void main(String[] args) {
    SingletonEnum.INSTANCE.doWork();
  }
}
```
Что именно является антипаттерном?
- Тестирование 
- Тяжело подложить мок
- Делает код менее гибким
- PowerMock очень медленный (cglib)
- Static getInstance

##### Синглтоны в спринге
- Синглтоны в спринге по умолчанию не ленивы (not `lazy`, they are `eager`)
- Fail fast (Обнарудить ошибку во время компиляции дешевле чем на запущенном проде)

Зачем же нужны ленивые синглтоны?
- Lazy injection - since 4.3
- @Lazy вместе с @Autowired
- Когда используейтся очень редкий сервис

Соблазн switch-case как пизанская башня
- какой может случиться баг в проде изза switch-case? - забытый break
