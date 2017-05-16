#### rt1
утверждают, что семёрка работает быстрее
так же традиционные улучшения типизации, полезным выглядит `Closure::call()` (ещё бы он мне не понравился)

бесполезным - `<=>`
(это просто ощущения. я не понимаю о чём я говорю, про php7 впервые услышал из тестового задания)

#### rt2
не берусь судить, на мой взгляд нет единого правильного подхода

- у ООП есть известные проблемы с моделированием предметной области и отладкой
- функциональное позволяет красиво и безопасно преобразовывать данные
- процедурное ближе к железу, из-за чего предсказуемее и проще в отладке

#### rt3
плюсы: отделение декларативных компонентов от императивных, представления от логики

подходит для монолитных веб-приложений (рендеринг на сервере), не применим для современных веб-приложений (рендеринг на клиенте)

#### rt4
использую express (роутинг, мидлварь) + sequelize (орм) + jade (шаблонизатор) + karma/protractor/jade (e2e тестирование).
местами напоминает RoR (за отсутствием других источников лучших практик)                                                                                              
                                                                                                                                                                      
не использую цельные фреймворки, поскольку в мире node с ними было печально, когда я изучал этот вопрос                                                               
                                                                                                                                                                      
хочу освоить **revel**                                                                                                                                                   
                                                                                                                                                                      
плюсы использования фреймворков:                                                                                                                                      
- не нужно думать о структуре приложения                                                                                                                              
- единообразный и предсказуемый дизайн компонентов

минусы:                                                                                                                                                               
- тяжеловестность и неспецифичность                                                                                                                                   
                                                                                                                                                                      
#### rt5                                                                                                                                                                   
плюсы шаблонизации:                                                                                                                                                   
- отделение кода от представления

минусы:                                                                                                                                                               
- попытка наследовать синтаксис от `xml` или `с` (jade лишен таких минусов:)                                                                                         
                                                                                                                                                                 
#### rt6                                                                                                                                                              
плюсы vсs:                                                                                                                                                       
- предсказуемость разработки, в первую очередь командной                                                                                                         
- устойчивость против утраты результатов разработки                                                                                                              
- возможность отследить автора вплоть до строки                                                                                                                  
- возможность настроить двойную ответственность (code-review)                                                                                                    
- возможность работать с удовольствием                                                                                                                           

большим челленджем была привязка состояния базы к состоянию кода без использования миграций

#### rt7
0. конструктивное общение (терпимость к друг-другу, нетерпимость к наплевательству и тупизне)
1. code-review
2. тестирование (не важно как, но готовый участок кода должен тестироваться весь)
3. автоматизация
4. соглашения

соглашения должны быть:
1. жесткими
2. автоматическими

#### rt8
- github issue tracker
- jira
- redmine (ужос-ужос)
- trello (каждый раз превращается в записную книжку)

плюсы трекера такие же, как у vсs

#### rt9
был мимолётный опыт с mongo и reddis

#### rt10
- javascript и его диалекты
- plsql
- однажды на java

-----

#### rp1
```bash
cat > _query.php << EOF
#!/usr/bin/php
<?php
\$query = stream_get_contents(STDIN);
\$conn = pg_connect("dbname=students");
\$result = pg_query(\$conn, \$query);
var_dump(json_encode(pg_fetch_all(\$result)));
EOF

echo "select name, grade from students inner join likes on (students.id = likes.liked_id) group by name, grade having count(likes.id) > 1;" | php _query.php

echo "select distinct students.* from students join likes on (students.id = likes.like_id) join students as likeless \
on (likes.liked_id = likeless.id) left outer join likes as dislikes on (likeless.id = dislikes.like_id) where dislikes.like_id is null;" | php _query.php

echo "select students.* from students left outer join likes as dislikes on (students.id = dislikes.like_id) \
left outer join likes as disliked on (students.id = disliked.liked_id) where dislikes.id is null and disliked.id is null;" | php _query.php
```

#### rp2
к сожалению в постгресе не воспроизводится.
имхо, такая задача решается добавлением таблички last_articles и триггера on insert, который подсовывает в неё id последней статьи и убирает четвёртую с конца.
просто потому что добавление статьи - не ресурсоёмкая задача, и вычисления логичнее производить в этой точке цикла.

какая именно тут предполагается оптимизация, я затрудняюсь угадать.
пробовал листать на PHP - дохнет, сделал с PDO - время исполнения на 10миллионах записей выросло на три порядка:
 - `ORDER BY id DESC`:    0m0.036s
 - `PDO::CURSOR_SCROLL`: 0m23.628s
```bash
cat > _cursor.php << EOF
#!/usr/bin/php
<?php
\$db = new PDO('pgsql:dbname=students');
\$st = \$db->prepare('select * from students order by id', array(PDO::ATTR_CURSOR => PDO::CURSOR_SCROLL));
\$st->execute();
\$result = array();
\$result[0] = \$st->fetch(PDO::FETCH_NUM, PDO::FETCH_ORI_LAST);
\$result[1] = \$st->fetch(PDO::FETCH_NUM, PDO::FETCH_ORI_PRIOR);
\$result[2] = \$st->fetch(PDO::FETCH_NUM, PDO::FETCH_ORI_PRIOR);
var_dump(json_encode(\$result));
EOF
```

#### rp3
```bash
cat > _geocoder_api.php << EOF
#!/usr/bin/php
<?php

\$input = readline("Address: ");

\$url = "https://maps.googleapis.com/maps/api/place/autocomplete/json?types=geocode&key=AIzaSyCR97ZTMWarr42Z5oHVaPo2VLgHJw_YnlI&input=".urlencode(\$input);
\$autocomplete = json_decode(file_get_contents(\$url));
\$predictions = [];

\$get_coordinates = function (\$place) {
  \$url = "http://maps.googleapis.com/maps/api/geocode/json?sensor=true?key=AIzaSyCR97ZTMWarr42Z5oHVaPo2VLgHJw_YnlI&address=".urlencode(\$place);
  \$tmp = json_decode(file_get_contents(\$url));
  return array(\$tmp->results[0]->formatted_address, json_encode(\$tmp->results[0]->geometry->location));
};
\$print_result = function (\$result) {
  printf("Full address: %s\n", \$result[0]);
  printf("Coordinates:  %s\n", \$result[1]);
};

if (!sizeof(\$autocomplete->predictions)) {
  echo "better luck next time!\n";
  exit;
}

if (sizeof(\$autocomplete->predictions) > 1) {
  foreach (\$autocomplete->predictions as \$record) {
    array_push(\$predictions, \$record->description);
    printf("[%u] %s\n", sizeof(\$predictions), \$record->description);
  }
  \$choise = (int) readline("chose an option from above: ");
  if (sizeof(\$predictions) >= \$choise && \$choise > 0) {
    \$address = \$predictions[\$choise-1];
    \$result = \$get_coordinates(\$address);
    \$print_result(\$result);
  } else {
    echo "wrong choise!\n";
    exit;
  }
} else {
  \$result = \$get_coordinates(\$autocomplete->predictions[0]->description);
  \$print_result(\$result);
}
EOF
```

#### rp4
```bash
cat > _fixture_a.json << EOF
{ "content": [-1, 0, 1, 2, 3, 4, 5, 6, 7] }
EOF

cat > _fixture_b.json << EOF
{ "content": [3, 4, 7, 8, 9, 10, 11, 12] }
EOF

cat > _compare.php << EOF
#!/usr/bin/php
<?php
if (array_key_exists(1, \$argv)) {
  \$array_a = json_decode(file_get_contents(\$argv[1]), true)["content"];
} else {
  \$input = readline("type in your array: ");
  \$array_a = json_decode(sprintf('[%s]', \$input));
}
if (array_key_exists(2, \$argv)) {
  \$array_b = json_decode(file_get_contents(\$argv[2]), true)["content"];
} else {
  \$input = readline("type in your array: ");
  \$array_b = json_decode(sprintf('[%s]', \$input));
}
\$result = array(
  "unique_to_a" => array_diff(\$array_a, \$array_b),
  "unique_to_b" => array_diff(\$array_b, \$array_a),
);
var_dump(json_encode(\$result));
EOF
```
```
php _compare.php _fixture_a.json _fixture_b.json
php _compare.php _fixture_a.json
php _compare.php
```
