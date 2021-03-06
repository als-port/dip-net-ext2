## Доработка 2, комментарии  
  
  
Прилагаю результаты второго этапа доработки по дипломному проекту. Подробные комментарии помогли понять ожидаемое (все же специфика не совсем моя пока, поэтому даже не сомневался). При этом, на мой взгляд, формулировку про репозиторий желательно немного уточнить/дополнить.  
Как Вы и писали про переход на локальные ресурсы, переложил работу приложения и системы CI/CD в кластер, развернутый при прохождении 12-го блока.  
В целом: было сделано два пайплайна, которые работают независимо (один по всем коммитам в main, второй по тегам в main).
  
Далее по пунктам:  
  
```text
Вы, как разработчик много и часто коммитите в репозиторий, для проверки правильности 
вашего коммита происходит сборка Docker-контейнера. Конечно, в настоящем CI этому будут 
предшествовать тесты, но будем считать, что факт сборки образа уже подтверждает 
правильность изменения. На этом этапе (запуск пайплайна по коммиту в main) пайплайн 
должен останавливаться на проверке (сразу после сборки образа). Можно отправить его в 
docker hub с тегом latest, чтобы разработчик имел возможность образ скачать к себе на 
компьютер и протестировать локально. Но в рамках дипломного проекта это не оязательно.  
```  
    
Ниже привожу картинки с отработкой пайплайна по коммитам. [Ссылка](https://github.com/als-port/dip-net-ext2/blob/main/jenkins/commit/Jenkinsfile) на Jenkinsfile.  
Логику, как по данному пункту, так и по следующему, перевел на вебхуки. Тригером, с помощью фильтра отлавливаются все коммиты в main. Далее - сборка, отправка docker hub с тегом latest.  
    
![Stages_commit](https://github.com/als-port/dip-net-ext2/blob/main/pics/any_commit_pipe_to_main_stages.png)  
  
  
Вывод консоли (последняя страница, остальные можно посмотреть в папке pics):  
  
![Cons_out_commit](https://github.com/als-port/dip-net-ext2/blob/main/pics/any_commit_pipe_to_main4.png) 
  
      
```text
Предположим, что локальное тестирование разработчика устроило, он считает, что текущий 
код в main хорош для релиза. Чтобы произвести релиз, разработчик создаёт тег в git (!) и 
отправляет его git push --tags. Тег имеет формат версии (например, 1.0.0). При появлении 
нового тега в git, Jenkins должен запустить другой пайплайн (или единственный пайплайн 
должен переключиться на другой режим). В этом пайплайне (запускающемся не по событию 
коммита, а по событию появления нового тега) следует сделать git checkout по имени этого 
тега, собрать образ, поставив его (образа) тег в значение равное имени тега в git. После 
этого его отправить в docker hub и запустить деплой именно этого (тегированного версией) 
образа в k8s. Если вы всё-таки не хотите использовать helm для такого деплоя (хотя сегодня 
это практически стандарт де-факто для индустрии), можете прямо в kube_app.yaml заменить 
тег ообраза на PLACEHOLDER и перед деплоем менять его, например, через 
sed: sed -i -e "s/PLACEHOLDER/${GIT_TAG}/" kube_app.yaml. Разумеется, переменную следует 
использовать правильную, ту, где у вас будет значение этого нового тега.  
```
  
Ниже привожу картинки с отработкой пайплайна по тегам. [Ссылка](https://github.com/als-port/dip-net-ext2/blob/main/jenkins/tags/Jenkinsfile) на Jenkinsfile.
Здесь чуть сложнее: вебхуком отлавливается факт тега в main, вычисляются коммит и ветка (может как-то проще можно было, но не хватает опыта), а далее в репозиторий образов и передеплой.    
      
![Stages_tags](https://github.com/als-port/dip-net-ext2/blob/main/pics/taged_pipe_ok_main_stages.png)  
  
  
Вывод консоли (последняя страница, остальные можно посмотреть в папке pics):  
  
![Cons_out_tags](https://github.com/als-port/dip-net-ext2/blob/main/pics/taged_pipe_ok_main4.png)  
  
  
Ниже картинка с результатом теста в другую - тестовую ветку.  
  
![fail_test_branch](https://github.com/als-port/dip-net-ext2/blob/main/pics/fail_tag_from_test_branch.png)  
  
  
  
