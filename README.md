# Odatav4. Quickstart.
Итак, начать делать сервис можно, как обычно, в транзакции `SEGW` или сделать сервис полностью вручную, а затем его опубликовать. Еще можно использовать RESTful Application Programming (RAP) Model, но последний подход не реализован в SAP в версии 750, поэтому здесь рассматриваться не будет.
# Начало.
## Создание сервиса через segw
В транзакции `SEGW` жмем **Create project** и заполняем название/описание, в project type выбираем загадочное 4, в generation strategy выбираем 1 - Standard OData 4.0.
![segw_create.png](https://github.com/Via04/odatav4-guide/blob/master/static/segw_create.png)

При выборе появится предупреждение с нотой.
![segw_warn.png](https://github.com/Via04/odatav4-guide/blob/master/static/segw_warn.png)

Суть в том, что нас предупреждают, что начиная с текущей версии (750) `SEGW` является legacy способом создания сервисов и лучше либо полностью с нуля реализовывать сервис (потому что таким образом можно полностью через eclipse разрабатывать сервис) либо через RAP. В нашем случае можно смело это предупреждение игнорировать, но тем не менее, ниже я расскажу, как создать сервис без `SEGW`.  
Итак, мы получаем сервис. Принципы работы с таким сервисом в основном схожи с odatav2. Главным отличием является создание Navigation Property. Теперь ассоциации исчезли и связь между типами устанавливается напрямую в Navigation Property, а тип связи контролируется двумя галочками Collection и Nullable (см. скриншот ниже).
![nav_prop.png](https://github.com/Via04/odatav4-guide/blob/master/static/nav_prop.png)

Из названия понятно за что они отвечают, но ниже будет показано подробно.  
Для любого типа независимо от его отношений, всегда создается EntitySet, а для любого Navigation Property всегда создается соответствующий Navigation Property Binding для EntitySet. В этом плане odatav4 можно рассматривать как язык программирования, по аналогии с ABAP, мы определяем тип, а затем создаем переменную в виде сущности этого типа.  
После того как сервис создали и сгенерировали, нужно опубликовать его в сервисную группу, а группу сделать доступной (аналогично тому, как это делается в /iwfnd/maint_service).
## Создание сервиса через /iwbep/v4_admin
Для того, чтобы создать сервис напрямую без `SEGW` и избежать предупреждения, нужно самому создать и реализовать два класса:
1. **Model** класс - наследуется от абстрактного `/IWBEP/CL_V4_ABS_MODEL_PROV` и нужен для создания модели данных oData (MPC по простому)
2. **Data provider** класс - наследуется от абстрактного `/IWBEP/CL_V4_ABS_DATA_PROVIDER` и нужен для описания бизнес логики работы сервиса (DPC по простому)

Классы \*MPC и \*DPC генерируемые `segw` ничем не отличаются, поэтому, в целом, можно уверенно использовать `segw` для создания сервисов. Но не все можно сделать в `segw` поэтому для унификации подхода к реализации возможно действительно лучше весь сервис целиком реализовывать через код. Подробней про работу `segw` с odatav4 и про реализацию классов для ручного создания сервиса будет в следующих параграфах.   
Теперь из этих двух классов нужно создать сервис - для этого переходим в транзакцию `/iwbep/v4_admin` и жмем кнопку **Register Service**. Вписываем имя сервиса, версию сервиса (для того, чтобы можно было иметь несколько реализаций \*DPC класса для версионности API), созданный класс-модель, созданный класс-провайдер, описание и пакет для сервиса. Далее нам сразу предложат определить сервис в группу либо определяем сразу, либо позже, нажав нет. Тогда сервис попадет в группу по-умолчанию (/iwbep/all).  
Подробней об этом в [следующем параграфе](#публикация-сервиса).
## Публикация сервиса.
Для того, чтобы опубликовать сервис нужны две транзакции:
- `/iwbep/v4_admin` - для создания групп и регистрации сервисов в группы (также через нее можно регистрировать сервисы созданные вручную).
- `/iwfnd/v4_admin` - для публикации групп (аналогично `/iwfnd/maint_service`).

Сервисы, которые только были созданы, попадают в **Default Service group /IWBEP/ALL**. Сами эти группы нужны для управления доступом к сервису, на запуск сервисов из каждой сервисной группы нужны полномочия объекта **S_START** с полями **R4TR G4BA group_id**.
Итак, у нас есть сервис, для ограничения доступа к нему (ну и предоставления, сервисы из группы ALL по-умолчанию не опубликованы) создадим свою группу в `/iwbep/v4_admin` с помощью кнопки **Register Group** (см. ниже на скриншоте).
![def_group.png](https://github.com/Via04/odatav4-guide/blob/master/static/def_group.png)

Теперь в свою группу определим сервис. Кнопка **Assign Service**.
![assign_serv.png](https://github.com/Via04/odatav4-guide/blob/master/static/assign_serv.png)

Вписываем созданную группу и имя сервиса, репозиторий в случае нашей системы (750) всегда будет DEFAULT, другие варианты соответственно для сервисов, разработанных через SADL и RAP.

Теперь созданную группу нужно опубликовать через `/iwfnd/v4_admin`. Жмем кнопку **Publish Service Groups**.
![pub_serv.png](https://github.com/Via04/odatav4-guide/blob/master/static/pub_serv.png)

- **System Alias** - аналогично `/iwfnd/maint_service`, в нашем случае почти всегда LOCAL
- **Service Group** - имя нашей группы

Вписываем и жмем кнопку **Get Service Groups**. На экране появятся подробности по группе, првоеряем, что все верно выделяем запись с группой и жмем кнопку **Publish Service Groups**:
![pub_serv_1.png](https://github.com/Via04/odatav4-guide/blob/master/static/pub_serv_1.png)

В появившемся окне вписываем имя публикации:
![assign_serv_2.png](https://github.com/Via04/odatav4-guide/blob/master/static/assign_serv_2.png)

И далее указываем запрос. В принципе, на этом этапе создание/публикация сервиса закончена и теперь сервис можно тестировать через `/iwfnd/gw_client`. Для того, чтобы быстро перейти в `/iwfnd/gw_client` для тестирования сервиса, вернемся на главный экран транзакции `/iwfnd/v4_admin`. Слева выберем наш новый сервис и нажмем кнопку **Service Implementation** на вкладке Available Services:
![nav_serv.png](https://github.com/Via04/odatav4-guide/blob/master/static/nav_serv.png)

Мы попадем в транзакцию `/iwbep/v4_admin` с уже выбранной нашей группой, жмем кнопку **Service Test** (в выпадающем списке можно выбрать где открыть в браузере или в GW_CLIENT, но по-умолчанию выбирается последний) на тулбаре сервиса, затем выбираем настроенный алиас и попадаем в классический `/iwfnd/gw_client` с уже вписанным URI для получения metadata сервиса. На этом создание сервиса завершено.
# oDatav4 framework
## oDatav4 Model
Реализация \*MPC классов в oDatav4 во многом похожа на oDatav2 поэтому я не буду здесь полностью расписывать как создать модель, наследуясь от класса `/IWBEP/CL_V4_ABS_MODEL_PROV`, а пройдусь только по неочевидным нюансам. К тому же, почти наверняка, модель всегда будет создаваться из транзакции `segw`. Но для общего понимания, а также для решения проблем, которые в `segw` нерешаемы, написан это параграф.  
Итак для создания модели, отнаследовавшись от класса `/IWBEP/CL_V4_ABS_MODEL_PROV` нужно переопределить метод `define`. Во входящем параметре `io_model` будут все необходимые методы для описания модели. Хороший пример реализации модели можно посмотреть в стандартном демо классе `/iwbep/cl_v4_tea_busi_pr_model`. Рассмотрим пару неочевидных примеров.
### Сущность для прикрепления документов
В oDatav2 для прикрепления документов был специальный тип сущности EDM.Stream (галка media в segw), он же есть и в odatav4, однако entity type типа EDM.Stream создать в odatav4 нельзя, зато можно создать отдельно поле типа EDM.Stream. Допустим мы создали Complex Type *Attachment* в `segw`:
![attach_type.png](https://github.com/Via04/odatav4-guide/blob/master/static/attach_type.png)

По-умолчанию в `segw` нет типа поля EDM.Stream, но его можно указать вручную внутри кода для модели. Код из имплементации метода `/iwbep/if_v4_mp_basic~define`:
```abap
DATA lo_prop TYPE REF TO /iwbep/if_v4_med_prim_prop.
DATA(lo_complex_type) = io_model->get_complex_type( iv_complex_type_name = 'ATTACHMENT' ).
lo_prop ?= lo_complex_type->get_property( iv_property_name = 'CONTENT' ).
lo_prop->set_content_type_property( 'FILETYPE' ).
lo_prop->set_file_name_property( 'FILENAME' ).
lo_prop->set_value_control_property( 'VAL_CTL' ).
lo_prop->set_edm_type( /iwbep/if_v4_med_element=>gcs_edm_data_types-stream ).
lo_prop->set_is_nullable( ).
lo_prop ?= lo_complex_type->get_property( 'FILETYPE' ).
lo_prop->set_is_technical( ).
lo_prop ?= lo_complex_type->get_property( 'FILENAME' ).
lo_prop->set_is_technical( ).
lo_prop ?= lo_complex_type->get_property( 'VAL_CTL' ).
lo_prop->set_is_technical( ).
lo_prop ?= lo_complex_type->get_property( 'LENGTH' ).
lo_prop->set_is_technical( ).
```
Здесь из модели получается созданный ранее в \*MPC комлексный тип **ATTACHMENT**, затем из него достается поле **CONTENT**. Для этого поля указываются технические поля, где будет длина файла, имя контрольный флаг для удаления и mime тип. Далее для каждого технического поля указывается явно, что оно техническое, чтобы они не мелькали в теле запроса для сущности с данным комплексным типом.  
Теперь данный комплексный тип надо прикрепить к *Entity type*:
```abap
 lo_complex_prop = lo_entity_type->create_complex_property( 'ATTACHMENT' ).
 lo_complex_prop->set_edm_name( 'Attachment' ).             "#EC NOTEXT
 lo_complex_prop->set_complex_type( 'ATTACHMENT' ).
 ```
Тут `lo_entity_type` это объект типа сущности, полученный из `io_model`. Для прикрепления файла к сущности, созданной таким образом, надо сначала создать саму сущность, передав в теле запроса в json все необходимое, а затем сделав patch заполнить отдельно **ATTACHMENT** (это можно сделать одним http запросом c помощью batch).  
Необязательно делать именно так через комплексный тип, можно и для обычной сущности указать поля как для комплексного типа, и, тогда для прикрепления можно использовать классический POST. Главная цель данного примера было показать, как реализовать работу с файлами на уровне модели.  
Пример обработки файлов отправленных таким способом можно подсмотреть все там же в демо классах SAP выполнив например запрос `/sap/opu/odata4/iwbep/tea/default/iwbep/tea_busi_product/0001/Products(4)/ProductPicture/Picture` и передав в теле запроса какую-нибудь картинку (тест кейс номер 378 в v4_test) в `gw_client`. Реализация модели: `/iwbep/cl_v4_tea_busi_pr_model->define_ct_picture`, реализация дата провайдера:
`/iwbep/cl_v4_tea_busi_pr_data->update_entity_product`.  
Однако данный пример не совсем верен, с точки зрения обработки http запросов, так как в классической реализации put запрос (на который и настроен саповский тест кейс) должен полностью перезаписывать целевую сущность, и правильным в данном случае было бы использовать patch (так как картинка это лишь одно поле всей сущности). Но это только лишний раз демонстрирует гибкость фреймворка.
### Немного о Navigation Property
То как мы поставим две галки **Collection** и **Nullable** в `segw` определяет, что будет передано в следующий метод:
```abap
lo_nav_prop->set_target_multiplicity(
    iv_target_multiplicity = /iwbep/if_v4_med_element=>gcs_med_nav_multiplicity-to_one ).
```
Соответственно возможные варианты (все они в `/iwbep/if_v4_med_element=>gcs_med_nav_multiplicity`):
- 1..1 - `/iwbep/if_v4_med_element=>gcs_med_nav_multiplicity-to_one`
- 1..\[0..1\] - `/iwbep/if_v4_med_element=>gcs_med_nav_multiplicity-to_one_optional`
- 1..\* - `/iwbep/if_v4_med_element=>gcs_med_nav_multiplicity-to_many_optional`
Из названия понятно, как они работают, ниже в параграфе про Data provider, будет показано, как происходит общение между сущностями связанными таким образом.
## oDatav4 Data provider
Прежде, чем мы перейдем к примерам кода и реализации сервиса, сначала важно понять общий принцип работы фреймворка, поэтому сначала в двух словах я расскажу, как устроен абстрактный класс `/IWBEP/CL_V4_ABS_DATA_PROVIDER` отвечающий за реализацию логики сервиса.  
Фреймворк состоит из 3 уровней-интерфейсов, которые вызываются последовательно друг из друга и на каждом уровне реализация все менее "абстрактная".
* `/IWBEP/IF_V4_DP_BASIC` - Самый базовый user-friendly интерфейс, который подходит в 90% процентов случаев. Все операции CRUD, а также `READ_REF_TARGET_KEY` для expand (подробности дальше)
* `/IWBEP/IF_V4_DP_INTERMEDIATE` - Переопределенные методы отсюда позволяют создать свою реализацию expand (только для READ операций) аналог GET_EXPANDED_ENTITY в odatav2, обработка PATCH, обработка etag (концепт блокировки на уровне api, подобно саповским блокировкам)
* `/IWBEP/IF_V4_DP_ADVANCED` - Самый низкоуровневый интерфейс, любой запрос сначала попадает в него, содержит все те же CRUD методы, однако переализация методов этого интерфейса прерывает стандартную обработку фреймворка, позволяя таким образом например в переопределении метода `create_entity` создавать deep entity по аналогии с odatav2. Хотя классический концепт фреймворка предполагает, что создание, чтение и обновление сущностей по каскаду будет производиться с помощью их *basic* методов, а для операций над каскадами (expand) будет использоваться метод `READ_REF_TARGET_KEY` с заданием соотвествующих ключей для создания, чтения или обновления дочерних сущностей, для оптимизации можно переопределить соответствующие методы *advanced* интерфейса.  
Итак, фреймворк odatav4 работает следующим образом сначала в абстрактном классе `/IWBEP/CL_V4_ABS_DATA_PROVIDER` решается, какой метод CRUD был вызван и в зависимости от этого вызывается соответствующий метод *advanced* интерфейса, затем если тот метод не переопределен, проводится стандартная обработка запроса и вызываются более высокоуровневые методы соответственно *intermediate* и *basic* интерфейса.  
### Basic interface
#### /IWBEP/IF_V4_DP_BASIC~READ_ENTITY
Сразу начнем с примеров, так как по нюансам расписывать можно бесконечно. Допустим у нас есть сущность и мы хотим читать ее по ключу, GET запрос следующего вида:
`/zhcm_ref_recommend_srv/0001/Recommendations(тут_ид)`. Для обработки запроса подобного вида фреймворк попытается вызвать метод `/IWBEP/IF_V4_DP_BASIC~READ_ENTITY`, переопределив данный метод, сделаем реализацию вызова соответствующего обработчика (при создании через `segw` автоматически cгенерируется метод `recommendationse_read` и подставится его вызов в переопределенный `/IWBEP/IF_V4_DP_BASIC~READ_ENTITY`, но для понимания работы я пишу так, будто `segw` ничего не генерил):
```abap
DATA lv_entityset_name TYPE /iwbep/if_v4_med_element=>ty_e_med_internal_name.

io_request->get_entity_set( IMPORTING ev_entity_set_name = lv_entityset_name ).

CASE lv_entityset_name.
*-------------------------------------------------------------------------*
*             EntitySet -  Recommendations
*-------------------------------------------------------------------------*
	WHEN 'RECOMMENDATIONSET'.
*     Call the entity set generated method
  	recommendationse_read(
       EXPORTING io_request  = io_request
                 io_response = io_response
    ).
    WHEN OTHERS.
      super->/iwbep/if_v4_dp_basic~read_entity( io_request  = io_request
                                                io_response = io_response ).
ENDCASE.
```
Здесь смотрится для какого сета был вызван запрос, и в кейсе, в зависимости от входящего сета выбирается обработчик запроса. Посмотрим теперь непосредственно на реализацию этого метода.
```abap
  METHOD recommendationse_read.
    DATA ls_done_list TYPE /iwbep/if_v4_requ_basic_read=>ty_s_todo_process_list.
    DATA lo_msgs      TYPE REF TO /iwbep/if_v4_message_container.
    DATA: BEGIN OF ls_key,
            rec_id TYPE zehcm_ats_rec_id,
          END OF ls_key.

    io_request->get_todos( IMPORTING es_todo_list = DATA(ls_todos) ).
    IF ls_todos-process-key_data = abap_true.
      io_request->get_key_data( IMPORTING es_key_data = ls_key ).
      ls_done_list-key_data = abap_true.
    ENDIF.
    mo_handler->get_recommendations( EXPORTING iv_key  = ls_key-rec_id
                                     IMPORTING et_data = DATA(lt_recommendations)
                                     CHANGING  co_msgs = lo_msgs ).
    DATA(lo_msg_hdl) = CAST /iwbep/cl_v4_message_container( lo_msgs ).
    lo_msg_hdl->get_messages( IMPORTING et_message = DATA(lt_msgs) ).
    IF line_exists( lt_msgs[ message_type = 'E' ] ).
      RAISE EXCEPTION TYPE  /iwbep/cx_gateway
        EXPORTING message_container = lo_msgs.
    ENDIF.
    TRY.
        DATA(ls_busi_data) = lt_recommendations[ 1 ].
      CATCH cx_sy_itab_line_not_found.
    ENDTRY.
    io_response->set_busi_data( is_busi_data = ls_busi_data ).
    io_response->set_is_done( is_todo_list = ls_done_list ).
  ENDMETHOD.
```
Любая обработка oData запроса начинается с получения TODO флагов, которые содержат флаги того, что требуется получить в запросе, является ли у нас запрос запросом по ключу, требуется ли полностью сущность или нет, нужен ли expand, нужно ли вернуть только конкретные поля в ответной сущности, есть ли фильтры в запросе и многое другое - все это указывается в получаемой с помощью `get_todos` структуре. Затем по условиям, в зависимости от того, что требуется в запросе происходит обработка. В данном случае, если ключ в запросе был передан, то мы получаем его в структуру ls_key (сам вид структуры должен содержать все необходимые поля с таким же именем полей, как внутренние имена ключевых полей соответствующей сущности). Затем уже передаем данный ключ в конечный обработчик (который уже никак не связан с oData фреймворком и здесь для читаемости и удобства) и получаем ответ, ошибки если они есть помещаются в `/iwbep/if_v4_message_container`, специальный контейнер, который может содержать в себе несколько exception и сообщений. Объект данного контейнера можно получить с помощью класса `/iwbep/cl_v4_message_container`. Затем проверяется, если в контейнере есть ошибки, то вызывается исключение `/iwbep/cx_gateway`, аналог *busi_exception* в odatav2, в которое передается контейнер с ошибками. При этом в ошибке в ответе на веб-запрос будет видно все ошибки помещенные в данный контейнер, в противовес odatav2, где можно было показать только одну ошибку. Ну и конечный шаг, мы передаем в ответ на веб-запрос результат  с помощью `io_response->set_busi_data( is_busi_data = ls_busi_data ).` и устанавливаем обработанные флаги. **Если какой-либо флаг был передан во входящем запросе, но не был установлен как выполненный, то веб-запрос выдаст ошибку, что функционал реализован не полностью**. То есть например если был передан в веб запросе $select=RecId, то в todo флагах флаг select будет 'X', если при этом в ls_done_list этот флаг не будет установлен как 'X', то запрос будет считаться невыполненным и выдаст ошибку. **Также `set_is_done` всегда должен быть последним вызванным методом фреймворка**. Если сначала вызвать `set_is_done`, а потом определить `set_busi_data`, **то будет ошибка**. 
#### /IWBEP/IF_V4_DP_BASIC~READ_ENTITY_LIST с фильтром по связанной сущности
Аналогично *READ_ENTITY* для чтения набора по фильтру, либо без ключа служит метод `/IWBEP/IF_V4_DP_BASIC~READ_ENTITY_LIST`.  
Для него также аналогично требуется обработка в зависимости от имени входящего сета, но так как она в общем-то ничем не отличается от обработки в случае read, то перейду сразу к примеру конечной реализации для конкретного сета. Запрос GET `/zhcm_ref_recommend_srv/0001/RequestLists?$filter=(Bukrs eq '1010' and contains(Detail/Descr, 'cleaner'))`. Как видно здесь мы запрашиваем список, отфильтрованный по bukrs и содержащий в описании связанной сущности слово cleaner. Перейдем к обработчику:
```abap
METHOD requestlistset_read_list.
    DATA ls_done            TYPE /iwbep/if_v4_requ_basic_list=>ty_s_todo_process_list.
    DATA lt_descr_range     TYPE zcl_hcm_ref_rec_mpc_ext=>ty_t_descr_range.
    DATA lt_bukrs_range     TYPE zcl_hcm_ref_rec_mpc_ext=>ty_t_bukrs_range.
    DATA lt_city_range      TYPE zcl_hcm_ref_rec_mpc_ext=>ty_t_city_range.
    DATA lt_func_napr_range TYPE zcl_hcm_ref_rec_mpc_ext=>ty_t_func_napr_range.
    DATA lt_sllvl_range     TYPE zcl_hcm_ref_rec_mpc_ext=>ty_t_sllvl_range.

    io_request->get_todos( IMPORTING es_todo_list = DATA(ls_features) ).
    IF ls_features-process-top = abap_true.
      io_request->get_top( IMPORTING ev_top = DATA(lv_top) ).
      ls_done-top = abap_true.
    ENDIF.
    IF ls_features-process-skip = abap_true.
      io_request->get_skip( IMPORTING ev_skip = DATA(lv_skip) ).
      ls_done-skip = abap_true.
    ENDIF.
    IF ls_features-process-filter = abap_true.
      " get filterable props and filters for them
      io_request->get_filter_props_with_ranges( IMPORTING et_property_path = DATA(lt_requested_filters) ).
      LOOP AT lt_requested_filters INTO DATA(lv_prop).
        CASE lv_prop.
          WHEN 'DETAIL-ZR_DESCR'.
            io_request->get_filter_ranges_for_prop( EXPORTING iv_property_path = lv_prop
                                                    IMPORTING et_range         = lt_descr_range ).
          WHEN 'BUKRS'.
            io_request->get_filter_ranges_for_prop( EXPORTING iv_property_path = lv_prop
                                                    IMPORTING et_range         = lt_bukrs_range ).
          WHEN 'CITY'.
            io_request->get_filter_ranges_for_prop( EXPORTING iv_property_path = lv_prop
                                                    IMPORTING et_range         = lt_city_range ).
          WHEN 'FUNC_NAPR'.
            io_request->get_filter_ranges_for_prop( EXPORTING iv_property_path = lv_prop
                                                    IMPORTING et_range         = lt_func_napr_range ).
          WHEN 'SLLVL'.
            io_request->get_filter_ranges_for_prop( EXPORTING iv_property_path = lv_prop
                                                    IMPORTING et_range         = lt_sllvl_range ).
        ENDCASE.
      ENDLOOP.
      ls_done-filter = abap_true.
    ENDIF.
    DATA(lo_global_msgs) = io_response->get_message_container( ).
    mo_handler->get_requestlist( EXPORTING iv_top             = lv_top
                                           iv_skip            = lv_skip
                                           it_bukrs_range     = lt_bukrs_range
                                           it_city_range      = lt_city_range
                                           it_descr_range     = lt_descr_range
                                           it_func_napr_range = lt_func_napr_range
                                           it_sllvl_range     = lt_sllvl_range
                                 IMPORTING et_requestlist     = DATA(lt_busi_data)
                                           ev_count           = DATA(lv_count)
                                 CHANGING  co_msgs            = lo_global_msgs ).
    DATA(lo_msg_hdl) = CAST /iwbep/cl_v4_message_container( lo_global_msgs ).
    lo_msg_hdl->get_messages( IMPORTING et_message = DATA(lt_msgs) ).
    IF line_exists( lt_msgs[ message_type = 'E' ] ).
      RAISE EXCEPTION TYPE  /iwbep/cx_gateway
        EXPORTING message_container = lo_global_msgs.
    ENDIF.
    io_response->set_busi_data( it_busi_data = lt_busi_data ).
    io_response->set_count( iv_count = lv_count ).
    io_response->set_is_done( is_todo_list = ls_done ).
  ENDMETHOD.
```
Итак, получаем todo лист, если было запрошено конкретное количество записей, то записываем их в переменные lv_top и lv_skip и сразу устанавливаем нужные флаги, затем если был запрошен фильтр, получаем фильтр по возможным полям в соответствующие ренджы. Фильтр можно также получить и в виде osql строки, которую сразу можно использовать как where условие в цикле или селект с помощью `io_request->get_filter_osql_where_clause( IMPORTING ev_osql_where_clause = DATA(lv_filter_clause) )`, но в данном случае такой подход не подходил, так как данные собирались уж слишком из разных мест. После вызывается соответствующий обработчик, обрабатываются сообщения, устанавливается результат и количество получаемых записей (которое можно получит в веб-запросе с помощью флага $count=true), а также обработанные флаги. Получим ответ такого вида:
```json
{
  "@odata.context" : "$metadata#RequestLists",
  "@odata.metadataEtag" : "W/\"20240925153024\"",
  "value" : [
    {
      "Objid" : "70000001",
      "Name" : "Специалист по договорной работе",
      "Begda" : "2020-10-20",
      "City" : "1234",
      "CityT" : "Город",
      "Bukrs" : "1010",
      "Butxt" : "ООО Рога и копыта",
      "FuncDetermination" : "11",
      "FuncDeterminationT" : "Практика SAP",
      "Sllvl" : "17"
    }
  ]
}
```
Здесь нужно обратить внимание, на то что хоть мы и фильтровали по описанию, но самого описания в выходной сущности нет, для получения описания нужно сделать соответствующий expand на detail, в котором и содержится это описание. Такая конструкция возможна, благодаря тому, что сущности связаны как 1..1, при этом в коде **обработка фильтра происходит на уровне корневой сущности запроса**, а фреймворк сам проверяет, допустимое ли это условия фильтра или нет в зависимости от связей между сущностями.
#### /IWBEP/IF_V4_DP_BASIC~CREATE_ENTITY
Итак теперь перейдем к созданию сущностей, причем сущности можно создавать/читать/удалять каскадами, определяя ключ с помощью метода `READ_REF_TARGET_KEY` (об этом далее), а можно переопределить низкоуровневый `/IWBEP/IF_V4_DP_ADVANCED~CREATE_ENTITY`, и написать реализацию обработку deep entity там, не вызывая обработку по каскаду. Но сначала рассмотрим самый базовый пример для закрепления основ работы с фреймворком.  
POST `zhcm_ref_recommend_srv/0001/Recommendations` с телом запроса:
```json
{
      "RecId" : "0050569C11E11EEF98F83F955FF11E7D",
      "RecDate" : "2024-08-26",
      "RecStatus" : "POT_1",
      "ZrObjid" : "80000104",
      "Nachn" : "Ващенко",
      "Vorna" : "Иван",
      "Midnm" : "Тестовый",
      "Email" : "VIA0408@GMAIL.COM",
      "Phone" : "+79773146587",
      "Comnt" : "хороший человек",
      "Pernr" : "",
      "Bonus" : 0.00,
      "CvUrl" : "",
      "CvName" : "",
      "Attachment" : {

      }
    }
```
Attachment в данном случае не нужен, так как выше в модели мы смотрели - все поля в Attachment либо *технические*, либо *nullable*. Тем не менее здесь в json он оставлен, чтобы было более лучшее представление с какой сущностью мы работаем.
Посмотрим на переопределенный метод `/IWBEP/IF_V4_DP_BASIC~CREATE_ENTITY`:
```abap
 DATA lv_entityset_name TYPE /iwbep/if_v4_med_element=>ty_e_med_internal_name.

  io_request->get_entity_set( IMPORTING ev_entity_set_name = lv_entityset_name ).

  CASE lv_entityset_name.
*-------------------------------------------------------------------------*
*             EntitySet -  Recommendations
*-------------------------------------------------------------------------*
    WHEN 'RECOMMENDATIONSET'.
*     Call the entity set generated method
      recommendationse_create(
           EXPORTING io_request  = io_request
                     io_response = io_response
                       ).
    WHEN OTHERS.
      super->/iwbep/if_v4_dp_basic~create_entity( io_request  = io_request
                                                  io_response = io_response ).
  ENDCASE.
```
Здесь ничего не обычного, просто вызываем нужный обработчик в зависимости от сущности.  
В методе-обработчике в общем-то тоже ничего необычного. Все то, что мы рассмотрели выше в [параграфе про read_entity](#/IWBEP/IF_V4_DP_BASIC~READ_ENTITY), но только наоборот:
```abap
  METHOD recommendationse_create.
    DATA ls_done_list TYPE /iwbep/if_v4_requ_basic_create=>ty_s_todo_process_list.
    DATA ls_data      TYPE zcl_hcm_ref_rec_mpc_ext=>ts_recommendation.

    io_request->get_todos( IMPORTING es_todo_list = DATA(ls_features) ).
    io_request->get_busi_data( IMPORTING es_busi_data = ls_data ).
    DATA(lo_global_msgs) = io_response->get_message_container( ).
    IF ls_features-process-busi_data = abap_true.
      ls_done_list = mo_handler->set_recommendation( CHANGING cs_recommendation = ls_data
                                                              co_msgs           = lo_global_msgs ).
    ENDIF.
    DATA(lo_msg_hdl) = CAST /iwbep/cl_v4_message_container( lo_global_msgs ).
    lo_msg_hdl->get_messages( IMPORTING et_message = DATA(lt_msgs) ).
    IF line_exists( lt_msgs[ message_type = 'E' ] ).
      RAISE EXCEPTION TYPE  /iwbep/cx_gateway
        EXPORTING message_container = lo_global_msgs.
    ENDIF.
    io_response->set_busi_data( is_busi_data = ls_data ).
    io_response->set_is_done( is_todo_list = ls_done_list ).
  ENDMETHOD.
```
Главное отличие здесь, что теперь мы сначала вызываем `io_request->get_busi_data` для получения данных с фронта, а затем передаем это в обработчик. Также здесь другой todo лист, который мы рассмотрим подробней ниже внутри метода `mo_handler->set_recommendation`.  
Полный код метода здесь приводиться не будет, чтобы не усложнять восприятие. Только последние 2 строчки которые нам и нужны для понимания:
```abap
rs_result = VALUE #( busi_data         = abap_true
                     partial_busi_data = abap_true ).
```
Итак здесь помимо флага обработанных данных в *done_list* пишется также флаг того, что данные обработаны в неполном объеме. **Это нужно, так данные для Attachment не был переданы в теле запроса, а Attachment это комплексный тип внутри Recommendation**.  
Сам же attachment заполняется следующим запросом PATCH (можно и PUT эту ситуацию можно обрабатывать по желанию) `/zhcm_ref_recommend_srv/0001/Recommendations(' RecId ')/Attachment/Content`, где в теле запроса и находится файл, а в заголовках его длина, MIME тип и название. Причем можно отправить за 1 запрос и json и файл с помощью batch запроса, но в данном случае файл прикрепляется в зависимости от ответа с бекенда, поэтому такой подход не подходит. Подробней про метод `/IWBEP/IF_V4_DP_ADVANCED~UPDATE_ENTITY` в параграфе по [advanced interface](#Advanced-interface).  
#### /iwbep/if_v4_dp_basic~read_ref_target_key_data
Данный метод вызывается всегда для обработки каскадов данных. Например запрос:
```http
GET /zhcm_ref_recommend_srv/0001/RequestLists?$expand=Detail&$filter=(contains(Detail/Descr, 'cleaner'))
```
Обработчик данного запроса, но без expand мы смотрели выше. Теперь же, когда есть expand с точки зрения работы изначального обработчика для RequestLists ничего не поменяется, он также будет вызван, но после этого класс `/IWBEP/CL_V4_ABS_DATA_PROVIDER` вызывает метод `/iwbep/if_v4_dp_basic~read_ref_target_key_data` (или `/iwbep/if_v4_dp_basic~read_ref_target_key_data_list` если связь со связанной сущностью `gcs_med_nav_multiplicity-to_many_optional`), цель которого определить ключ для дальнейшего вызова *read_entity*, но уже с **Detail** сущностью. Рассмотрим реализацию данного метода:
```abap
  METHOD /iwbep/if_v4_dp_basic~read_ref_target_key_data.
    DATA lv_source_entity_name TYPE /iwbep/if_v4_med_element=>ty_e_med_internal_name.

    io_request->get_source_entity_type( IMPORTING ev_source_entity_type_name = lv_source_entity_name ).

    CASE lv_source_entity_name.
      WHEN zcl_hcm_ref_rec_mpc_ext=>mc_entity_types-request_list.
        set_common_key_detail( io_request  = io_request
                               io_response = io_response ).
      WHEN zcl_hcm_ref_rec_mpc_ext=>mc_entity_types-recommendation.
        set_common_key_req( io_request  = io_request
                            io_response = io_response ).
      WHEN OTHERS.
        super->/iwbep/if_v4_dp_basic~read_ref_target_key_data( io_request  = io_request
                                                               io_response = io_response ).
    ENDCASE.
  ENDMETHOD.
```
Здесь у нас снова кейс, где в зависимости от того из какой сущности вызван expand вызывается соответствующий обработчик. Рассмотрим нашу реализацию для **Detail**:
```abap
  METHOD set_common_key_detail.
    DATA: BEGIN OF ls_key,
            zr_objid TYPE hrobjid,
          END OF ls_key.
    DATA ls_done_list TYPE /iwbep/if_v4_requ_basic_ref_l=>ty_s_todo_process_list.

    io_request->get_todos( IMPORTING es_todo_list = DATA(ls_todo_list) ).
    IF ls_todo_list-process-source_key_data = abap_true.
      io_request->get_source_key_data( IMPORTING es_source_key_data = ls_key ).
      ls_done_list-source_key_data = abap_true.
    ENDIF.
    io_response->set_target_key_data( is_target_key_data = ls_key ).
    io_response->set_is_done( is_todo_list = ls_done_list ).
  ENDMETHOD.
```
В данном случае все просто, мы читаем ключ той сущности из которой вызван expand и для **Detail** назначаем тот же ключ, заполняя, уже по классике, todo лист. **Таким образом, дальше `/IWBEP/CL_V4_ABS_DATA_PROVIDER` вызовет снова read_entity для expand сущности, а в качестве ключа в нее попадет то, что мы сейчас записали в этом методе.** Используя данный подход, можно унифицровать обработку deep entity, сделав 1 обработчик для чтения/записи сущностей, независимо от того с deep entity запрос или нет.
### Advanced interface
Несмотря на то, что подход выше можно использовать и для создания сущностей, все же зачастую гораздо быстрее (и с точки зрения оптимизации и с точки зрения написания), когда вся deep entity записывается, что называется, "за раз". В таком случае на помощь приходит метод `/iwbep/if_v4_dp_advanced~create_entity`.
#### /iwbep/if_v4_dp_advanced~create_entity
Принцип работы с данным методом схож с аналогичным в *basic* интерфейсе, но главное отличие здесь во времени вызова, данный метод вызывается первым внутри `/IWBEP/CL_V4_ABS_DATA_PROVIDER`. Также в нем больше возможностей взаимодействия с запросом, рассмотрим пример ниже для создания deep entity. Запрос:
```http
POST /iwbep/tea/default/iwbep/tea_busi/0001/TEAMS
```
Тело запроса достаточно большое и не играет роли для данного примера, но его можно посмотреть в тест кейсе V4_TEST номер 128.
Data provider:
```abap
METHOD /iwbep/if_v4_dp_advanced~create_entity.

    DATA: lv_entity_type_name TYPE /iwbep/if_v4_med_element=>ty_e_med_internal_name,
          ls_todo_list        TYPE /iwbep/if_v4_requ_adv_create=>ty_s_todo_list,
          lo_navigation_node  TYPE REF TO /iwbep/if_v4_navigation_node.



    io_request->get_todos( IMPORTING es_todo_list = ls_todo_list ).

    IF ls_todo_list-process-navigation EQ abap_false.
      io_request->get_entity_type( IMPORTING ev_entity_type_name = lv_entity_type_name ).
    ELSE.
      lo_navigation_node = io_request->get_navigation_path( ).
      DO.
        lo_navigation_node = lo_navigation_node->get_next_navigation_node( ).
        IF lo_navigation_node->is_navigation_target( ) EQ abap_true.
          lo_navigation_node->get_entity_type_name( IMPORTING ev_entity_type_name = lv_entity_type_name ).
          EXIT.
        ENDIF.
      ENDDO.
    ENDIF.


    CASE lv_entity_type_name.
      WHEN gcs_entity_type_names-employee.
        create_entity_employee(
          EXPORTING
            io_request  = io_request
            io_response = io_response ).

      WHEN gcs_entity_type_names-team.
        IF  ( ls_todo_list-process-target_key_data = abap_true )
        AND ( ls_todo_list-process-deep_busi_data = abap_true ).
          RAISE EXCEPTION TYPE /iwbep/cx_v4_tea
            EXPORTING
              http_status_code = /iwbep/cx_v4_tea=>gcs_http_status_codes-sv_not_implemented
              textid           = /iwbep/cx_v4_tea=>not_implemented.
        ENDIF.

        IF ( ls_todo_list-process-deep_busi_data = abap_true ).
          create_entity_team_deep(
            EXPORTING
              io_request  = io_request
              io_response = io_response ).
        ELSE.
          create_entity_team(
            EXPORTING
              io_request  = io_request
              io_response = io_response ).
        ENDIF.

      WHEN OTHERS.
        super->/iwbep/if_v4_dp_advanced~create_entity(
                 io_request  = io_request
                 io_response = io_response ).

    ENDCASE.

  ENDMETHOD.
  ```
  Мы получаем из URI имя сущности, на которую запрашивается CREATE, а затем в зависимости от того была ли передана deep entity в теле запроса или нет вызывается нужный обработчик. В нашем случае это `create_entity_team_deep`. Сама реализация метода достаточно большая и ее нет смысла рассматривать здесь, главная задача этого метода зполнить структуру вида:
```abap
TYPES:
BEGIN OF ty_s_team_and_manager.
  INCLUDE TYPE /iwbep/s_v4_tea_team.
TYPES team_2_manager TYPE /iwbep/s_v4_tea_manager.
TYPES END OF ty_s_team_and_manager .
```
Тут сама структура представляет сущность **Team**, а поле team_2_manager сущность **Manager** в целом данный подход к deep entity похожна аналогичный в odatav2. Последним примером покажу обработку update_entity с чтением header запроса.
#### /iwbep/if_v4_dp_advanced~update_entity.
Запрос PATCH/PUT `zhcm_ref_recommend_srv/0001/Recommendations(' RecId ')/Attachment/Content`.  

Любая обработка начинается с выбора обработчика, но в данном случае он только один:
```abap
  METHOD /iwbep/if_v4_dp_advanced~update_entity.
    DATA lv_entity_type_name TYPE /iwbep/if_v4_med_element=>ty_e_med_internal_name.

    io_request->get_entity_type( IMPORTING ev_entity_type_name = lv_entity_type_name ).

    CASE lv_entity_type_name.
      WHEN zcl_hcm_ref_rec_mpc_ext=>mc_entity_types-recommendation.
        recommendation_update( io_request  = io_request
                               io_response = io_response ).
      WHEN OTHERS.
        RAISE EXCEPTION TYPE /iwbep/cx_v4_tea
          EXPORTING textid = /iwbep/cx_v4_tea=>not_implemented.

    ENDCASE.
  ENDMETHOD.
```
Посмотрим код самого обработчика:
```abap
  METHOD recommendation_update.
    DATA ls_todos             TYPE /iwbep/if_v4_requ_adv_update=>ty_s_todo_list.
    DATA ls_done_list         TYPE /iwbep/if_v4_requ_adv_update=>ty_s_todo_process_list.
    DATA lt_provided_property TYPE /iwbep/if_v4_runtime_types=>ty_t_property_path.
    DATA lo_request_pro       TYPE REF TO /iwbep/cl_v4_request_info_pro.
    DATA lo_request_info      TYPE REF TO /iwbep/if_v4_request_info.
    DATA: BEGIN OF ls_rec_key,
            rec_id TYPE zshcm_ref_recommendation-rec_id,
          END OF ls_rec_key.
    DATA lv_media_value_provided   TYPE abap_bool.
    DATA lv_value_control_provided TYPE abap_bool.
    DATA ls_provided_property      TYPE /iwbep/if_v4_runtime_types=>ty_property_path.
    DATA ls_rec_req                TYPE zshcm_ref_recommendation.

    io_request->get_todos( IMPORTING es_todo_list = ls_todos ).
    DATA(lo_global_msgs) = io_response->get_message_container( ).
    IF ls_todos-process-key_data = abap_true.
      io_request->get_key_data( IMPORTING es_key_data = ls_rec_key ).
    ENDIF.

    IF ls_todos-process-patch = abap_false.
      RETURN.
    ENDIF.
    lo_request_pro ?= io_request.
    lo_request_info = lo_request_pro->get_base_request_info( ).
    DATA(lv_filename_header) = lo_request_info->get_http_header( 'content-disposition' ).
    FIND FIRST OCCURRENCE OF REGEX 'filename\=\"(\w+\.\w+)\"' IN lv_filename_header SUBMATCHES DATA(lv_filename).
    DATA(lv_length) = lo_request_info->get_http_header( 'content-length' ).
    io_request->get_provided_properties( IMPORTING et_provided_property_path = lt_provided_property ).

    LOOP AT lt_provided_property INTO ls_provided_property.
      CASE ls_provided_property.
        WHEN zcl_hcm_ref_rec_mpc_ext=>mc_attachment_property_path-content.
          lv_media_value_provided = abap_true.
        WHEN zcl_hcm_ref_rec_mpc_ext=>mc_attachment_property_path-content_ctl.
          lv_value_control_provided = abap_true.
      ENDCASE.
    ENDLOOP.

    IF ls_todos-process-busi_data = abap_true.
      io_request->get_busi_data( IMPORTING es_busi_data = ls_rec_req ).
    ENDIF.
    ls_rec_req-attachment-filename = lv_filename.
    ls_rec_req-attachment-length   = lv_length.
    IF     lv_media_value_provided        = abap_true
       AND lv_value_control_provided      = abap_true
       AND ls_rec_req-attachment-val_ctl <> /iwbep/if_v4_runtime_types=>gcs_value_control-indicate_null.
      mo_handler->add_attachment( EXPORTING iv_key            = ls_rec_key-rec_id
                                            is_attachment     = ls_rec_req-attachment
                                  IMPORTING es_recommendation = DATA(ls_recommendation)
                                            es_todo           = ls_done_list
                                  CHANGING  co_msgs           = lo_global_msgs ).
    ENDIF.
    io_response->set_busi_data( is_busi_data = ls_recommendation ).
    io_response->set_is_done( is_todo_list = ls_done_list ).
  ENDMETHOD.
```
Тут стоит, обратить внимание на lo_request_pro и lo_request_info.  
Класс `/iwbep/cl_v4_request_info_pro` является суперклассом для любого io_request и таким содержит методы всех уровней фреймворка (плюс дополнительные, один из которых затем сразу и используется). Объект данного класса можно получить из обработчиков любого из интерфейсов (*basic*, *intermediate* или *advanced*) и его цель предоставить тот функционал, который редко нужен и обычный фреймворк его не покрывает. Конкретно в данном примере с помощью объекта данного класса получаются данные по запросу и читаются заголовки по имени из которых достается имя прикрепленного файла. Оставшийся код уже так или иначе рассмотрен выше и не представляет интереса для обсуждения.
## Нюансы
Несмотря на удобство фреймворка, конкретно в нашей системе (750) у него есть недоработки. Главная о которой стоит упомянуть, так как с ней столкнутся все, кто будут пытаться перенести запрос с сервисом в другую систему, заключается в неправильном создании ключей для переноса настроечных таблиц, содержащих записи о созданном сервисе. При создании/регистрации группы сервисов/сервиса в запись о переносе добавляется 3 объекта:
| Описание           | ИД   | Тип  | Имя таблицы      |
| :----------------- | :--- | :--- | :--------------- |
| Содержимое таблицы | R3TR | TABU | /IWFND/C_V4_MSGR | 
| Содержимое таблицы | R3TR | TABU | /IWFND/C_V4_MSGT |
| Содержимое таблицы | R3TR | TABU | /IWFND/C_V4_RSAG |

В таблицу */IWFND/C_V4_MSGT* не записывается **sprsl** для русского языка, и нужно добавлять ключ вручную, а в таблицу /IWFND/C_V4_RSAG ключ пишется не полностью, нужно добавить "\*" в поле **assignment_id** ключа переноса, без этого запрос не будет переноситься и упадет с ошибкой. 
