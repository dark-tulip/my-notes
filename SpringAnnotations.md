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

```