# bitrix
# Тут буду выкладывать нестандартные, а иногда и стандартные кейсы по работе с 1с-Битрикс

+ Конвертер jpeg, gif, png в webp
  > Конечно под битрикс существует хороший бесплатные модуль [Вот ссылка](https://marketplace.1c-bitrix.ru/solutions/dev2fun.imagecompress/), но есть особенности что нужно подключать сторонние библиотеки на стороне сервера. Но не всегда есть такая возможность
  
```
class Webp
     {
      public static function make($src): string
      {
          $newImgPath = false;

          if ($src && function_exists('imagewebp')) {
              $newImgPath = str_replace(array('.jpg', '.jpeg', '.gif', '.png'), '.webp', $src);
              if (!file_exists($_SERVER['DOCUMENT_ROOT'].$newImgPath)) {
                  $info = getimagesize($_SERVER['DOCUMENT_ROOT'].$src);
                  if ($info !== false && ($type = $info[2])) {
                      switch ($type) {
                          case IMAGETYPE_JPEG:
                              $newImg = imagecreatefromjpeg($_SERVER['DOCUMENT_ROOT'].$src);
                              break;
                          case IMAGETYPE_GIF:
                              $newImg = imagecreatefromgif($_SERVER['DOCUMENT_ROOT'].$src);
                              break;
                          case IMAGETYPE_PNG:
                              $newImg = imagecreatefrompng($_SERVER['DOCUMENT_ROOT'].$src);
                              imagepalettetotruecolor($newImg);
                              imagealphablending($newImg, true);
                              imagesavealpha($newImg, true);
                              break;
                      }
                      if ($newImg) {
                          imagewebp($newImg, $_SERVER['DOCUMENT_ROOT'].$newImgPath, 90);
                          imagedestroy($newImg);
                      }
                  }
              }
          }

          return $newImgPath;
      }
    }
```
+ Получение всех профилей юридических лиц
> Мало ли у кого пропадут все профили юр.лиц и если есть dev то можно воспользоваться вот таким скриптом. Получаем все профили, потом уже по желанию. Например сохраняем их в csv файл и перебираем ,и с помощью [вот этого метода добавляем](https://dev.1c-bitrix.ru/api_help/sale/classes/csaleorderuserprops/csaleorderuserprops__add.110a5a48.php)

```
$db_sales = CSaleOrderUserProps::GetList(
    array("DATE_UPDATE" => "DESC"),
    array("PERSON_TYPE_ID" => 2)
);
$res = [];
while ($ar_sales = $db_sales->Fetch())
{
    $res[] = $ar_sales;
}
```

+ Если по каким-то обстоятельствам не работает капча и скрытые поля то идем сюда
> Решение еще конечно топорное, в дальнейшем надо будет добавить регулярное выражение ,но пока что времени увы нет
> Подключаем в init или создаем папку в php_interface например events и там создаем какой нибудь файл (не забываем что нам нужно подключить файл в init)
```
AddEventHandler("iblock", "OnBeforeIBlockElementAdd","тут будет название вашей функции");
```
> Далее сам скрипт
```
function blockSpam(&$arFields) {
    
    if($arFields["IBLOCK_ID"] == проверка на инфоблок){
        if (strpos($arFields["TEXT например"], 'https') !== false || дальше через или){
            global $APPLICATION;
            $APPLICATION->throwException("К сожалению, Ваш комментарий не может быть опубликован.");
            return false;
        }
    }
```
+ Подключение классов в init (для себя чтобы всегда было под рукой)
```
use \Bitrix\Main\Loader;

Loader::registerAutoLoadClasses(null, array(
    'Webp' => '/bitrix/php_interface/classes/Webp.php',
));
```
+ **Не работает после обновления ЛК в решениях АСПРО**
> Ищем глобальным поиском по проекту или через grep "что ищем" /где , вот такую строчку **$APPLICATION->reinitPath();**, меняем на
```
$APPLICATION->sDocPath2 = GetPagePath(false, true);
$APPLICATION->sDirPath = GetDirPath($APPLICATION->sDocPath2)
```
# Чуть чуть про ускорение сайта, наверное будет больше про (LCP)
> Событие **DOMContentLoaded** ленивая загрузка изображений. Можно конечно через **loading=lazy** но еще есть какие то проблемы у сафари. Что нам нужно, подкинуть js скрипт, добавить класс lazy, добивить два дата атрибута data-src and data-srcset.
```
document.addEventListener("DOMContentLoaded", function() {
    var lazyImages = [].slice.call(document.querySelectorAll("img.lazy"));;

    if ("IntersectionObserver" in window && "IntersectionObserverEntry" in window && "intersectionRatio" in window.IntersectionObserverEntry.prototype) {
        let lazyImageObserver = new IntersectionObserver(function(entries, observer) {
            entries.forEach(function(entry) {
                if (entry.isIntersecting) {
                    let lazyImage = entry.target;
                    lazyImage.src = lazyImage.dataset.src;
                    lazyImage.srcset = lazyImage.dataset.srcset;
                    lazyImage.classList.remove("lazy");
                    lazyImageObserver.unobserve(lazyImage);
                }
            });
        });

        lazyImages.forEach(function(lazyImage) {
            lazyImageObserver.observe(lazyImage);
        });
    }
});
