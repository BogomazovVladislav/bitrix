# bitrix
# Тут буду выкладывать нестандартные, а иногда и стандартные кейсы по работе с 1с-Битрикс

+ Конвертер jpeg, gif, png в webp
  > Конечно под битрикс существует хороший бесплатные модуль [Вот ссылка][https://marketplace.1c-bitrix.ru/solutions/dev2fun.imagecompress/], но есть особенности что нужно подключать сторонние библиотеки на стороне сервера. Но не всегда есть такая возможность
  
`class Webp
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
    }`
