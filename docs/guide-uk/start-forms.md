Робота з формами
================

В даному розділі ми обговоримо отримання даних від користувачів. На сторінці буде розміщена форма з полями, де можна буде вказати ім’я та email. Отримані дані будуть зображені на сторінці для їх підтвердження.

Для того, щоб досягти дану ціль, крім створення [події](structure-controllers.md) і двох [представлень](structure-views.md)
ви створите [модель](structure-models.md).

В даному керівництві ви дізнаєтесь:

* Як створити [модель](structure-models.md) для даних, вказаних користувачем;
* Як оголосити правила перевірки переданих даних;
* Як створити HTML форму в [представлені](structure-views.md).


Створення моделі <a name="creating-model"></a>
---------------------------------------------

У файлі `models/EntryForm.php` створіть клас моделі `EntryForm` як показано нижче. Він буде використовуватись для зберігання даних, вказаних користувачем. Детальніше про присвоєння імен файлам класів читайте в розділі
«[Автозавантаження класів](concept-autoloading.md)».

```php
<?php

namespace app\models;

use yii\base\Model;

class EntryForm extends Model
{
    public $name;
    public $email;

    public function rules()
    {
        return [
            [['name', 'email'], 'required'],
            ['email', 'email'],
        ];
    }
}
```

Даний клас розширює клас [[yii\base\Model]], який є складовою частиною фреймворка і зазвичай використовується для роботи з даними форм.

Клас містить 2 публічні властивості `name` і `email`, які використовуються для зберігання даних, вказаних користувачем.
Він також містить метод `rules()`, який повертає набір правил поведінки даних. Правила, оголошені в коді вище означають наступне:

* Поля `name` і `email` обов’язкові для заповнення;
* Поле `email` повино містити правильну адресу email.

Якщо об’єкт `EntryForm` заповнений даними користувача, то для їх перевірки ви можете викликати метод
[[yii\base\Model::validate()|validate()]]. У випадку невдалої перевірки властивість [[yii\base\Model::hasErrors|hasErrors]]
дорівнюватиме `true`. За допомогою [[yii\base\Model::getErrors|errors]] можна дізнатись, які саме виникли помилки.


Створення події <a name="creating-action"></a>
------------------------------------------------

Далі створіть подію `entry` в контролері `site`, точно так, як ви робили це раніше.

```php
<?php

namespace app\controllers;

use Yii;
use yii\web\Controller;
use app\models\EntryForm;

class SiteController extends Controller
{
    // ...існуючий код...

    public function actionEntry()
    {
        $model = new EntryForm;

        if ($model->load(Yii::$app->request->post()) && $model->validate()) {
            // дані в $model успішно провірені

            // робимо щось корисне з $model ...
 
            return $this->render('entry-confirm', ['model' => $model]);
        } else {
            // або сторінка відображається вперше, або ж є помилка в даних
            return $this->render('entry', ['model' => $model]);
        }
    }
}
```

Подія створює об’єкт `EntryForm`. Потім вона намагається заповнити модель даними із масива `$_POST`, доступ
до якого забеспечує Yii за допомогою [[yii\web\Request::post()]]. Якщо модель успішно заповнена, тобто користувач відправив дані з HTML форми, то для перевірки даних буде викликаний метод [[yii\base\Model::validate()|validate()]].

Якщо все гаразд, подія зобразить представлення `entry-confirm`, яке покаже користувачу вказані ним дані.
В іншому випадку буде зображено представлення `entry`, яке містить HTML форму і помилки перевірки даних, якщо вони є.

> Інформація: `Yii::$app` являє собою глобально доступний екземпляр-одинак
[додатка](structure-applications.md) (singleton). Одночасно це [Service Locator](concept-service-locator.md),
який надає доступ до компонентів типу `request`, `response`, `db` і так далі. В коді выще для доступу до даних з `$_POST`
був використаний компонент `request`.


Створення представлення <a name="creating-views"></a>
----------------------------------------------------

В завершення, створюємо два представлення з іменами `entry-confirm` і `entry`, котрі зображаються подією `entry` з минулого підрозділа.

Представлення `entry-confirm` просто зображає ім’я та email. Воно мусить бути збережене у файлі `views/site/entry-confirm.php`.

```php
<?php
use yii\helpers\Html;
?>
<p>Ви вказали наступну інформацію:</p>

<ul>
    <li><label>Name</label>: <?= Html::encode($model->name) ?></li>
    <li><label>Email</label>: <?= Html::encode($model->email) ?></li>
</ul>
```

Представлення `entry` відображає HTML форму. Воно мусить бути збережене у файлі `views/site/entry.php`.

```php
<?php
use yii\helpers\Html;
use yii\widgets\ActiveForm;
?>
<?php $form = ActiveForm::begin(); ?>

    <?= $form->field($model, 'name') ?>

    <?= $form->field($model, 'email') ?>

    <div class="form-group">
        <?= Html::submitButton('Надіслати', ['class' => 'btn btn-primary']) ?>
    </div>

<?php ActiveForm::end(); ?>
```

Для побудови HTML форми представлення використовує потужний [віджет](structure-widgets.md) [[yii\widgets\ActiveForm|ActiveForm]].
Методи `begin()` і `end()` виводять відкриваючий і закриваючий теги форми. Між цими викликами створюються поля для заповнення за допомогою метода [[yii\widgets\ActiveForm::field()|field()]]. Першим іде поле для "name", другим — для "email".
Далі для генерації кнопки відправлення даних викликається метод [[yii\helpers\Html::submitButton()]].


Спробуєм <a name="trying-it-out"></a>
--------------------------------------

Щоб побачити все, що було створено під час роботи, відкрийте в браузері наступний URL:

```
http://hostname/index.php?r=site/entry
```

Ви побачите сторінку з формою і двома полями для заповнення. Перед кожним полем є надпис, який вказує, яку саме
інформацію слід вказувати. Якщо ви натиснете на кнопку відправлення даних без самих даних або якщо вкажете email в невірному
форматі, то ви побачите повідомлення з помилкою біля кожного проблемного поля.

![Форма з помилками](../guide/images/start-form-validation.png)

Після введення вірних даних і їх відправки, ви побачите сторінку з даними, які щойно вказали.

![Підтвердження введених даних](../guide/images/start-entry-confirmation.png)



### Як працює вся ця «магія» <a name="magic-explained"></a>

Ви, більш за все, ставите питанням про те, як все ж ця HTML форма працює насправді і яким чином. Весь процес може здатися трохи магічним: те як зображаються підписи до полів, помилки перевірки даних при некоректному заповненні і те що все це відбувається без перезавантаження сторінки.

Так, провірка даних дійсно проходить на стороні клієнта за допомогою JavaScript і на стороні сервера.
[[yii\widgets\ActiveForm]] достатньо продуманий, щоб взяти правила перевірки, які ви оголосили в `EntryForm`,
перетворити їх в JavaScript код і використовувати його для проведення перевірки. На випадок, якщо в браузері буде вимкнено JavaScript валідація проходить і на стороні сервера, як показано в методі `actionEntry()`. Це дає впевненість в тому, що дані коректні за любих обставин.

Підписи для полів генеруються методом `field()`, на основі імен властивостей моделі. Наприклад, підпис `Name` генерується
для властивості `name`. Ви можете модифікувати підписи наступним чином:

```php
<?= $form->field($model, 'name')->label('Ваше ім’я') ?>
<?= $form->field($model, 'email')->label('Ваш Email') ?>
```

> Інформація: У Yii є велика кількість віджетів, які дозволяють швидко будувати складні і динамічні представлення.
  Як ви дізнаєтесь пізніше, розробляти нові віджети доволі просто. Багато із представленнь можна винести у віджети, щоб використовувати це повторно в інших частинах і тим самим спростити розробку в майбутньому.

Резюме <a name="summary"></a>
-----------------------------

В даному розділі ви випробували кожну частину шаблона проектування MVC. Ви дізналися як створювати класи моделей для опрацювання і перевірки даних вказаних користувачем.

Також, ви дізналися як отримати дані від користувача і як їх відобразити тому ж користувачу. Ця задача може займати багато часу в процесі розробки. Yii надає потужні віджети, які роблять задачу максимально простою.

В наступному розділі ви дізнаєтесь як працювати з базами даних, що необхідно в більшості додатків.
