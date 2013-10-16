Title: Изучаем матрицы трансформаций в CSS
----
Date: 2012-06-11 12:56:32
----
Lang: ru
----
Author: 
----
License: Creative Commons Attribution 3.0 Unported
----
License_url: http://creativecommons.org/licenses/by/3.0/
----
Text:

<ul>
	<li><a href="#intro">Введение</a></li>
	<li><a href="#whatisamatrix">Что такое матрица?</a></li>
	<li><a href="#coordinates">Трансформации и системы координат</a></li>
	<li><a href="#calculatingtransform">Расчет трансформации: математика матриц и векторов</a></li>
	<li><a href="#compoundtransforms">Составные трансформации с помощью матриц</a></li>
</ul>

<h2 id="intro">Введение</h2>

<p>Матричные функции — <code>matrix()</code> and <code>matrix3d()</code> — две самые головоломные в плане понимания вещи в <a href="http://dev.w3.org/csswg/css3-transforms/">CSS3-трансформациях</a>. В большинстве случаев, ради простоты и ясности, вы будете пользоваться функциями вроде <code>rotate()</code> и <code>skewY()</code>. Но всё же за каждой трансформацией скрывается эквивалентная матрица. Полезно хоть слегка понимать, как они работают, так что давайте взглянем.</p>

<p>CSS-трансформации «растут» из линейной алгебры и геометрии. Хотя продвинутая математическая подготовка будет весьма не лишней, понять матричные функции можно и без нее. Но вы должны быть хорошо знакомы с CSS-трансформациями. Иначе — читайте про <a href="http://dev.opera.com/articles/view/css3-transitions-and-2D-transforms/">эффекты переходов CSS3 и 2D-трансформации</a>.</p>

<p>В этой статье я охвачу как матрицы 3 на 3, используемые для двумерных трансформаций, так и матрицы 4 на 4 для трехмерных.</p>

<p>Заметьте, что на момент этой публикации Opera не поддерживает трехмерных трансформаций. Я включила двумерный <code>matrix()</code>-эквивалент, где возможно.</p>

<p>В этой статье я также пользуюсь беспрефиксными версиями свойств <code>transform</code>. На практике эти свойства всё еще экспериментальны и могут измениться. Пока они не утверждены окончательно, добавляйте в свой CSS-код версии с префиксом (напр., <code>-o-transform</code>).</p>

<h2 id="whatisamatrix">Что такое матрица?</h2>

<p><strong><a href="http://ru.wikipedia.org/wiki/%D0%9C%D0%B0%D1%82%D1%80%D0%B8%D1%86%D0%B0_%28%D0%BC%D0%B0%D1%82%D0%B5%D0%BC%D0%B0%D1%82%D0%B8%D0%BA%D0%B0%29">Матрица</a></strong> — это прикольный математический термин для <q>прямоугольного массива чисел, символов или выражений</q> (см. <a href="#figure1">рис. 1</a>). У матриц множество математических и научных применений. Физики, например, используют их при изучении квантовой механики. В области компьютерной графики они используются для вещей типа — внезапно! — линейных трансформаций и проекции трехмерных изображений на двумерный экран. Это и есть то, что делают матричные функции: <strong><code>matrix()</code></strong> позволяет нам создавать линейные трансформации, а <strong><code>matrix3d()</code></strong>дает возможность создавать иллюзию трехмерности в двух измерениях с помощью CSS.</p>

<div id="figure1">
	<p><img src="http://dev.opera.com/articles/view/understanding-the-css-transforms-matrix/css-transforms-matrix1.gif" width="620" height="200" alt="Сетка с цифрами 3 на 3: верхний ряд 1, 2, 8; средний ряд: 10, 3, 9; нижний ряд: 7, 4, 0" /></p>
	<p class="caption">Рис. 1. Пример матрицы</p>
</div>

<p>Мы не будем далеко забредать в глубины продвинутой алгебры. Вы должны быть знакомы с <a href="http://ru.wikipedia.org/wiki/%D0%9F%D1%80%D1%8F%D0%BC%D0%BE%D1%83%D0%B3%D0%BE%D0%BB%D1%8C%D0%BD%D0%B0%D1%8F_%D1%81%D0%B8%D1%81%D1%82%D0%B5%D0%BC%D0%B0_%D0%BA%D0%BE%D0%BE%D1%80%D0%B4%D0%B8%D0%BD%D0%B0%D1%82">декартовой системой координат</a>. Можете также освежить в памяти, <a href="http://ru.wikipedia.org/wiki/%D0%A3%D0%BC%D0%BD%D0%BE%D0%B6%D0%B5%D0%BD%D0%B8%D0%B5_%D0%BC%D0%B0%D1%82%D1%80%D0%B8%D1%86">как перемножать матрицы и векторы</a> (либо воспользуйтесь <a href="http://www.bluebit.gr/matrix-calculator/">калькулятором</a>, типа предлагаемого Bluebit.gr).</p>

<p>Важный для понимания момент — то, что трансформация умножает матрицу на координаты точки (или точек), выраженные в виде вектора.</p>

<h2 id="coordinates">Трансформации и системы координат</h2>

<p>Сначала поговорим о системах координат. Каждая область просмотра документа является системой координат. Левый верхний угол — ее начало, с координатами (0,0). Значения увеличиваются вправо по оси X и вниз по оси Y. Ось Z определяет кажущееся расстояние до зрителя в случае <abbr title="трёхмерных">3D</abbr>-трансформаций. Бо́льшие значения — предметы ближе и крупнее, меньшие значения — мельче и дальше.</p>

<p>Когда трансформация применяется к объекту, она создает локальную систему координат. По умолчанию начало локальных координат — точка (0,0) — лежит в центре объекта, или на 50% ширины и 50% высоты (<a href="#figure2">рис. 2</a>).</p>

<div id="figure2">
	<p><img src="http://dev.opera.com/articles/view/understanding-the-css-transforms-matrix/css-transforms-matrix2.gif" width="620" height="420" alt="Пример локальной системы координат" /></p>
	<p class="caption">Рис.2. Локальная система координат</p>
</div>

<p>Мы можем изменить начало локальной системы координат подгонкой свойства transform-origin (<a href="#figure3">рис. 3</a>). Задание <code>transform-origin: 50px 70px;</code>, например, помещает начало координат в 50 пикселях от левого края объекта и в 70 пикселях от его верха. Трансформации каждой точки в локальной системе координат объекта рассчитываются относительно этого начала.</p>

<div id="figure3">
	<p><img src="http://dev.opera.com/articles/view/understanding-the-css-transforms-matrix/css-transforms-matrix4.gif" width="620" height="420" alt="Пример локальной системы координат" /></p>
	<p class="caption">Рис. 3. Локальная система координат, с началом в точке (50px,70px). Также показана точка (30px,30px).</p>
</div>

<p>Браузеры делают за вас эти вычисления каждый раз, когда вы применяете трансформацию. Вам нужно лишь знать, какие аргументы могут помочь достичь нужного вам эффекта.</p>

<h2 id="calculatingtransform">Расчет трансформации: математика матриц и векторов</h2>

<p>Взглянем на пример с использованием <a href="http://www.w3.org/TR/SVG/coords.html#TransformMatrixDefined">матрицы 3 на 3</a> для расчета двумерной трансформации (<a href="#figure4">рис. 4</a>). <a href="http://www.w3.org/TR/css3-transforms/#mathematical-description">Матрица 4 на 4</a>, используемая для трехмерных трансформаций, работает так же, с дополнительными числами для добавочной оси Z.</p>

<div id="figure4">
	<p><img src="http://dev.opera.com/articles/view/understanding-the-css-transforms-matrix/css-transforms-matrix3.gif" width="620" height="200" alt="Сетка с цифрами 3 на 3. Верхний ряд: a, c, e; средний ряд: b, d, f; нижний ряд: 0, 0, 1" /></p>
	<p class="caption">Рис. 4. Матрица двумерной CSS-трансформации</p>
</div>

<p>Мы можем записать это как <code>transform: matrix(a,b,c,d,e,f)</code>, где буквы от <code>a</code> до <code>f</code> — числа, определяемые типом трансформации, которую мы хотим применить. Матрицы — это рецепты тех видов трансформации, которые мы хотим применить. Это станет чуть понятнее, когда мы рассмотрим несколько примеров.</p>

<p>Когда мы применяем двумерную трансформацию, браузер умножает матрицу на вектор <strong>[x, y, 1]</strong>. Значения x и y — координаты конкретной точки в локальном пространстве координат.</p>

<p>Чтобы найти координаты после трансформации, мы умножаем каждый элемент каждой строки матрицы на соответствующую ему строку вектора. Затем складываем произведения (<a href="#figure5">рис. 5</a>).</p>

<div id="figure5">
	<p><img src="http://dev.opera.com/articles/view/understanding-the-css-transforms-matrix/css-transforms-matrix5.gif" width="620" height="200" alt="При умножении матрицы на вектор, результат это сумма результатов каждого элемента матрицы, помноженного на соответствующий элемент вектора" /></p>
	<p class="caption">Рис. 5. Умножение матрицы на вектор</p>
</div>

<p>Я знаю, что это выглядит как куча бессмысленных цифр и букв. Но, как отмечено выше, у каждого типа трансформаций — своя собственная матрица. <a href="#figure6">Рис. 6</a> показывает матрицу для трансформации сдвига.</p>

<div id="figure6">
	<p><img src="http://dev.opera.com/articles/view/understanding-the-css-transforms-matrix/css-transforms-matrix6.gif" width="620" height="200" alt="Матрица сдвига" /></p>
	<p class="caption">Рис. 6. Матрица сдвига</p>
</div>

<p>Значения <code>tx</code> и <code>ty</code> — значения, на которые должно быть сдвинуто начало координат. Мы также можем представить это с помощью вектора [1 0 0 1 tx ty]. Этот вектор служит аргументом для функции <code>matrix()</code>, как показано ниже.</p>

<pre><code>#mydiv{
   transform: matrix(1, 0, 0, 1, tx, ty);
}</code></pre>

<p>Давайте трансформируем объект, левый верхний угол которого совпадает с левым верхним углом области просмотра (<a href="#figure7">Рис. 7</a>). Его глобальные координаты равны (0,0).</p>

<div id="figure7">
	<p><img src="http://dev.opera.com/articles/view/understanding-the-css-transforms-matrix/css-transforms-matrix4b.png" width="620" height="384" alt="Объект с глобальными координатами (0,0)" /></p>
	<p class="caption">Рис. 7. Объект с глобальными координатами (0,0).</p>
</div>

<p>Мы переместим этот объект на 150 пикселей по осям x и y, используя начало координат трансформации по умолчанию. Ниже приведен CSS для этой трансформации.</p>

<pre><code>#mydiv{
   transform: matrix(1, 0, 0, 1, 150, 150);
}</code></pre>

<p>Кстати, это эквивалентно <code>transform: translate(150px,150px)</code>. Давайте рассчитаем результат этой трансформации для точки с координатами (220px,220px) (<a href="#figure8">Рис. 8</a>).</p>

<div id="figure8">
	<p><img src="http://dev.opera.com/articles/view/understanding-the-css-transforms-matrix/css-transforms-matrix7.gif" width="620" height="200" alt="Вычисление трансформации сдвига" /></p>
	<p class="caption">Рис. 8. Вычисление трансформации сдвига.</p>
</div>

<p>Трансформации задают соответствие координатам и расстояниям в локальной системе координат объекта предыдущей системе координат. То, где точка отобразится в области просмотра, зависит от примененного при трансформации сдвига от начальной точки объекта. В этом примере наша точка с координатами (220px,220px) теперь отображается в точке (370px,370px). Прочие координаты в границах нашего объекта тоже сместились на 150 пикселей вправо и на 150 пикселей вниз (<a href="#figure9">рис. 9</a>).</p>

<div id="figure9">
	<p><img src="http://dev.opera.com/articles/view/understanding-the-css-transforms-matrix/css-transforms-matrix7b.png" width="620" height="384" alt="Наш объект после применения трансформации" /></p>
	<p class="caption">Рис. 9. Наш объект после применения трансформации.</p>
</div>

<p>Матрица сдвига — особый случай. Она как <em>аддитивна</em>, так и <em>мультипликативна</em>. Более простым решением было бы просто прибавить значение сдвига к значениям <em>x</em><em>-</em> и <em>y</em>-координат нашей точки.</p>

<h3>Расчет трехмерной трансформации</h3>

<p>Выше мы рассмотрели матрицу переноса 3 на 3. Давайте возьмем другой пример, с использованием матрицы 4 на 4 для масштабирования (<a href="#figure10">рис. 10</a>).</p>

<div id="figure10">
	<p><img src="http://dev.opera.com/articles/view/understanding-the-css-transforms-matrix/css-transforms-matrix8.gif" width="620" height="200" alt="Матрица 4 на 4 для масштабирования" /></p>
	<p class="caption">Рис. 10. Матрица 4 на 4 для масштабирования.</p>
</div>

<p>Здесь <i>sx</i>, <i>sy</i> и <i>sz</i> представляют масштабные коэффициенты по каждой оси. С функцией <code>matrix3d</code> это примет такой вид: <code>transform: matrix3d(sx, 0, 0, 0, 0, sy, 0, 0, 0, 0, sz, 0, 0, 0, 0, 1)</code>.</p>

<p>Будем продолжать с тем же объектом, что раньше. Уменьшим его масштаб по осям x и y с помощью функции <code>matrix3d()</code>, как показано ниже.</p>

<pre><code>#mydiv{
    transform: matrix3d(.8, 0, 0, 0, 0, .5, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1);
}</code></pre>

<p>Это эквивалентно transform: scale3d(0.8, 0.5, 1). Поскольку мы масштабируем только по осям x и y (получая 2D-трансформацию), мы могли бы использовать также transform: matrix(.8, 0, 0, .5, 0, 0) либо scale(.8,.5). Результат трансформации виден на <a href="#figure11">рис. 11</a>.</p>

<div id="figure11">
	<p><img src="http://dev.opera.com/articles/view/understanding-the-css-transforms-matrix/css-transforms-matrix9b.png" width="620" height="430" alt="Объект 300 на 300 пикселей после применения трансформации масштабирования" /></p>
	<p class="caption">Рис. 11. Объект 300 на 300 пикселей после применения трансформации масштабирования.</p>
</div>

<p>Если умножить эту матрицу на координатный вектор [150,150,1] (<a href="#figure12">рис. 12</a>), мы получим такие новые координаты нашей точки: (120,75,1).</p>

<div id="figure12">
	<p><img src="http://dev.opera.com/articles/view/understanding-the-css-transforms-matrix/css-transforms-matrix9.gif" width="620" height="200" alt="Вычисление трансформации масштабирования" /></p>
	<p class="caption">Рис. 12: Вычисление трансформации масштабирования.</p>
</div>

<h3>Где взять значения матриц</h3>

<p>Значения матриц для каждой функции трансформации приведены как в спецификации <a href="http://www.w3.org/TR/SVG/coords.html#TransformMatrixDefined">Scalable Vector Graphics</a>, так и в спецификации <a href="http://www.w3.org/TR/css3-transforms/#mathematical-description">CSS Transforms</a>.</p>

<h2 id="compoundtransforms">Составные трансформации с помощью матриц</h2>

<p>Наконец, давайте рассмотрим, как создать составную трансформацию — трансформацию, эквивалентную применению нескольких функций трансформации одновременно. Ради простоты ограничимся двумя измерениями. Это значит, что мы будем использовать матрицу трансформации 3 на 3 и функцию matrix(). Этой трансформацией мы повернем наш объект на 45&#xB0; и увеличим его масштаб в 1,5 раза от исходного размера.</p>

<p>Матрица поворота, выраженная в виде вектора — <code>[cos(a) sin(a) -sin(a) cos(a) 0 0]</code>, где <i>a</i> — угол. Для масштабирования понадобится матрица <code>[sx 0 0 sy 0 0]</code>. Чтобы объединить их, умножим матрицу поворота на матрицу масштабирования, как показано на <a href="#figure13">рис. 13</a> (синус и косинус 45&#xB0; оба равны 0.7071).</p>

<div id="figure13">
	<p><img src="http://dev.opera.com/articles/view/understanding-the-css-transforms-matrix/css-transforms-matrix10.gif" width="620" height="200" alt="Вычисление матрицы составной трансформации" /></p>
	<p class="caption">Рис. 13: Вычисление матрицы составной трансформации.</p>
</div>

<p>В CSS это будет выглядеть так: <code>transform: matrix(1.0606, 1.0606, -1.0606, 1.0606, 0, 1)</code>. <a href="#figure14">Рис. 14</a> показывает результат после применения трансформации.</p>

<div id="figure14">
	<p><img src="http://dev.opera.com/articles/view/understanding-the-css-transforms-matrix/css-transforms-matrix10b.png" width="620" height="450" alt="Наш объект 300 на 300 пикселей после масштабирования и поворота" /></p>
	<p class="caption">Рис. 14: Наш объект 300 на 300 пикселей после масштабирования и поворота.</p>
</div>

<p>Теперь рассчитаем новые координаты в области просмотра для точки (298,110), как показано на <a href="#figure15">рис. 15</a>.</p>

<div id="figure15">
	<p><img src="http://dev.opera.com/articles/view/understanding-the-css-transforms-matrix/css-transforms-matrix11.gif" width="620" height="200" alt="Применение трансформации" /></p>
	<p class="caption">Рис. 15. Применение трансформации.</p>
</div>

<p>Новыми координатами нашей точки будут (199.393px,432.725px).</p>

<h2>Узнать больше</h2>

<p>Надеюсь, эта статья немного приподняла завесу тайны над CSS-трансформациями. Если ей это не удалось, попробуйте обратиться к ресурсам ниже.</p>

<ul>
	<li><a href="http://ru.wikipedia.org/wiki/%D0%9C%D0%B0%D1%82%D1%80%D0%B8%D1%86%D0%B0_%28%D0%BC%D0%B0%D1%82%D0%B5%D0%BC%D0%B0%D1%82%D0%B8%D0%BA%D0%B0%29">Матрица (математика)</a> в Википедии</li>
	<li><a href="http://www.senocular.com/flash/tutorials/transformmatrix/">Объяснение матрицы трансформации во Flash 8</a> (автор — Senocular)</li>
	<li><a href="http://mathworld.wolfram.com/topics/Transformations.html">Трансформации</a> на сайте WolframMathWorld</li>
</ul>

<p class="note">Текстура матрицы на обложке <a href="http://www.flickr.com/photos/zooboing/4335531769/">Патрика Хосли</a>.</p>