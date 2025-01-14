# 構造我們自己的 Types 和 Typeclasses

## Algebraic Data Types 入門

在前面的章節中，我們談了一些 Haskell 內建的型別和 Typeclass。而在本章中，我們將學習構造型別和 Typeclass 的方法。

我們已經見識過許多型別，如 `Bool`、`Int`、`Char`、`Maybe` 等等，不過在 Haskell 中該如何構造自己的型別呢？好問題，一種方法是使用 _data_ 關鍵字。首先我們來看看 `Bool` 在標準函式庫中的定義：

```haskell
data Bool = False | True
```

_data_ 表示我們要定義一個新的型別。`=` 的左端標明型別的名稱即 `Bool`，`=` 的右端就是_值構造子_ \(_Value Constructor_\)，它們明確了該型別可能的值。`|` 讀作"或"，所以可以這樣閲讀該聲明：`Bool` 型別的值可以是 `True` 或 `False`。型別名和值構造子的首字母必大寫。

相似，我們可以假想 `Int` 型別的聲明：

```haskell
data Int = -2147483648 | -2147483647 | ... | -1 | 0 | 1 | 2 | ... | 2147483647
```

![](img/caveman.png)

頭尾兩個值構造子分別表示了 `Int` 型別的最小值和最大值，注意到真正的型別宣告不是長這個樣子的，這樣寫只是為了便於理解。我們用省略號表示中間省略的一大段數字。

我們想想 Haskell 中圖形的表示方法。表示圓可以用一個 Tuple，如 `(43.1,55.0,10.4)`，前兩項表示圓心的位置，末項表示半徑。聽著不錯，不過三維向量或其它什麼東西也可能是這種形式！更好的方法就是自己構造一個表示圖形的型別。假定圖形可以是圓 \(Circle\) 或長方形 \(Rectangle\)：

```haskell
data Shape = Circle Float Float Float | Rectangle Float Float Float Float
```

這是啥，想想？`Circle` 的值構造子有三個項，都是 Float。可見我們在定義值構造子時，可以在後面跟幾個型別表示它包含值的型別。在這裡，前兩項表示圓心的坐標，尾項表示半徑。`Rectangle` 的值構造子取四個 `Float` 項，前兩項表示其左上角的坐標，後兩項表示右下角的坐標。

談到「項」 \(field\)，其實應為「參數」 \(parameters\)。值構造子的本質是個函數，可以返回一個型別的值。我們看下這兩個值構造子的型別聲明：

```haskell
ghci> :t Circle
Circle :: Float -> Float -> Float -> Shape
ghci> :t Rectangle
Rectangle :: Float -> Float -> Float -> Float -> Shape
```

Cool，這麼說值構造子就跟普通函數並無二致囉，誰想得到？我們寫個函數計算圖形面積：

```haskell
surface :: Shape -> Float
surface (Circle _ _ r) = pi * r ^ 2
surface (Rectangle x1 y1 x2 y2) = (abs $ x2 - x1) * (abs $ y2 - y1)
```

值得一提的是，它的型別聲明表示了該函數取一個 `Shape` 值並返回一個 `Float` 值。寫 `Circle -> Float` 是不可以的，因為 `Circle` 並非型別，真正的型別應該是 `Shape`。這與不能寫`True->False` 的道理是一樣的。再就是，我們使用的模式匹配針對的都是值構造子。之前我們匹配過 `[]`、`False` 或 `5`，它們都是不包含參數的值構造子。

我們只關心圓的半徑，因此不需理會表示坐標的前兩項：

```haskell
ghci> surface $ Circle 10 20 10
314.15927
ghci> surface $ Rectangle 0 0 100 100
10000.0
```

Yay，it works！不過我們若嘗試輸出 `Circle 10 20` 到控制台，就會得到一個錯誤。這是因為 Haskell 還不知道該型別的字元串表示方法。想想，當我們往控制台輸出值的時候，Haskell 會先呼叫 `show` 函數得到這個值的字元串表示才會輸出。因此要讓我們的 `Shape` 型別成為 Show 型別類的成員。可以這樣修改：

```haskell
data Shape = Circle Float Float Float | Rectangle Float Float Float Float deriving (Show)
```

先不去深究 _deriving_（派生），可以先這樣理解：若在 `data` 聲明的後面加上 `deriving (Show)`，那 Haskell 就會自動將該型別至于 `Show` 型別類之中。好了，由於值構造子是個函數，因此我們可以拿它交給 `map`，拿它不全呼叫，以及普通函數能做的一切。

```haskell
ghci> Circle 10 20 5
Circle 10.0 20.0 5.0
ghci> Rectangle 50 230 60 90
Rectangle 50.0 230.0 60.0 90.0
```

我們若要取一組不同半徑的同心圓，可以這樣：

```haskell
ghci> map (Circle 10 20) [4,5,6,6]
[Circle 10.0 20.0 4.0,Circle 10.0 20.0 5.0,Circle 10.0 20.0 6.0,Circle 10.0 20.0 6.0]
```

我們的型別還可以更好。增加加一個表示二維空間中點的型別，可以讓我們的 `Shape` 更加容易理解：

```haskell
data Point = Point Float Float deriving (Show)
data Shape = Circle Point Float | Rectangle Point Point deriving (Show)
```

注意下 `Point` 的定義，它的型別與值構造子用了相同的名字。沒啥特殊含義，實際上，在一個型別含有唯一值構造子時這種重名是很常見的。好的，如今我們的 `Circle` 含有兩個項，一個是 `Point` 型別，一個是 `Float` 型別，好作區分。`Rectangle` 也是同樣，我們得修改 `surface` 函數以適應型別定義的變動。

```haskell
surface :: Shape -> Float
surface (Circle _ r) = pi * r ^ 2
surface (Rectangle (Point x1 y1) (Point x2 y2)) = (abs $ x2 - x1) * (abs $ y2 - y1)
```

唯一需要修改的地方就是模式。在 `Circle` 的模式中，我們無視了整個 `Point`。而在 `Rectangle` 的模式中，我們用了一個嵌套的模式來取得 `Point` 中的項。若出於某原因而需要整個 `Point`，那麼直接匹配就是了。

```haskell
ghci> surface (Rectangle (Point 0 0) (Point 100 100))
10000.0
ghci> surface (Circle (Point 0 0) 24)
1809.5574
```

表示移動一個圖形的函數該怎麼寫？它應當取一個 `Shape` 和表示位移的兩個數，返回一個位於新位置的圖形。

```haskell
nudge :: Shape -> Float -> Float -> Shape
nudge (Circle (Point x y) r) a b = Circle (Point (x+a) (y+b)) r
nudge (Rectangle (Point x1 y1) (Point x2 y2)) a b = Rectangle (Point (x1+a) (y1+b)) (Point (x2+a) (y2+b))
```

簡潔明瞭。我們再給這一 `Shape` 的點加上位移的量。

```haskell
ghci> nudge (Circle (Point 34 34) 10) 5 10
Circle (Point 39.0 44.0) 10.0
```

如果不想直接處理 `Point`，我們可以搞個輔助函數 \(auxilliary function\)，初始從原點創建圖形，再移動它們。

```haskell
baseCircle :: Float -> Shape
baseCircle r = Circle (Point 0 0) r

baseRect :: Float -> Float -> Shape
baseRect width height = Rectangle (Point 0 0) (Point width height)
```

```haskell
ghci> nudge (baseRect 40 100) 60 23
Rectangle (Point 60.0 23.0) (Point 100.0 123.0)
```

毫無疑問，你可以把你的數據型別導出到模組中。只要把你的型別與要導出的函數寫到一起就是了。再在後面跟個括號，列出要導出的值構造子，用逗號隔開。如要導出所有的值構造子，那就寫個..。

若要將這裡定義的所有函數和型別都導出到一個模組中，可以這樣：

```haskell
module Shapes
( Point(..)
, Shape(..)
, surface
, nudge
, baseCircle
, baseRect
) where
```

一個 `Shape` \(..\)，我們就導出了 `Shape` 的所有值構造子。這一來無論誰導入我們的模組，都可以用 `Rectangle` 和 `Circle` 值構造子來構造 `Shape` 了。這與寫 `Shape(Rectangle,Circle)` 等價。

我們可以選擇不導出任何 `Shape` 的值構造子，這一來使用我們模組的人就只能用輔助函數 `baseCircle` 和 `baseRect` 來得到 `Shape` 了。`Data.Map` 就是這一套，沒有 `Map.Map [(1,2),(3,4)]`，因為它沒有導出任何一個值構造子。但你可以用，像 `Map.fromList` 這樣的輔助函數得到 `map`。應該記住，值構造子只是函數而已，如果不導出它們，就拒絶了使用我們模組的人呼叫它們。但可以使用其他返回該型別的函數，來取得這一型別的值。

不導出數據型別的值構造子隱藏了他們的內部實現，令型別的抽象度更高。同時，我們模組的使用者也就無法使用該值構造子進行模式匹配了。

## Record Syntax

OK，我們需要一個數據型別來描述一個人，得包含他的姓、名、年齡、身高、電話號碼以及最愛的冰淇淋。我不知你的想法，不過我覺得要瞭解一個人，這些資料就夠了。就這樣，實現出來！

```haskell
data Person = Person String String Int Float String String deriving (Show)
```

O~Kay，第一項是名，第二項是姓，第三項是年齡，等等。我們造一個人：

```haskell
ghci> let guy = Person "Buddy" "Finklestein" 43 184.2 "526-2928" "Chocolate"
ghci> guy
Person "Buddy" "Finklestein" 43 184.2 "526-2928" "Chocolate"
```

貌似很酷，就是難讀了點兒。弄個函數得人的某項資料又該如何？如姓的函數，名的函數，等等。好吧，我們只能這樣：

```haskell
firstName :: Person -> String
firstName (Person firstname _ _ _ _ _) = firstname

lastName :: Person -> String
lastName (Person _ lastname _ _ _ _) = lastname

age :: Person -> Int
age (Person _ _ age _ _ _) = age

height :: Person -> Float
height (Person _ _ _ height _ _) = height

phoneNumber :: Person -> String
phoneNumber (Person _ _ _ _ number _) = number

flavor :: Person -> String
flavor (Person _ _ _ _ _ flavor) = flavor
```

唔，我可不願寫這樣的程式碼！雖然 it works，但也太無聊了哇。

```haskell
ghci> let guy = Person "Buddy" "Finklestein" 43 184.2 "526-2928" "Chocolate"
ghci> firstName guy
"Buddy"
ghci> height guy
184.2
ghci> flavor guy
"Chocolate"
```

你可能會說，一定有更好的方法！呃，抱歉，沒有。

開個玩笑，其實有的，哈哈哈～Haskell 的發明者都是天才，早就料到了此類情形。他們引入了一個特殊的型別，也就是剛纔提到的更好的方法 -- _Record Syntax_。

```haskell
data Person = Person { firstName :: String
                     , lastName :: String
                     , age :: Int
                     , height :: Float
                     , phoneNumber :: String
                     , flavor :: String
                     } deriving (Show)
```

與原先讓那些項一個挨一個的空格隔開不同，這裡用了花括號 `{}`。先寫出項的名字，如 `firstName`，後跟兩個冒號\(也叫 Paamayim Nekudotayim，哈哈~\(譯者不知道什麼意思~囧\)\)，標明其型別，返回的數據型別仍與以前相同。這樣的好處就是，可以用函數從中直接按項取值。通過 Record Syntax，Haskell 就自動生成了這些函數：`firstName`, `lastName`, `age`, `height`, `phoneNumber` 和 `flavor`。

```haskell
ghci> :t flavor
flavor :: Person -> String
ghci> :t firstName
firstName :: Person -> String
```

還有個好處，就是若派生 \(deriving\) 到 `Show` 型別類，它的顯示是不同的。假如我們有個型別表示一輛車，要包含生產商、型號以及出場年份：

```haskell
data Car = Car String String Int deriving (Show)
```

```haskell
ghci> Car "Ford" "Mustang" 1967
Car "Ford" "Mustang" 1967
```

若用 Record Syntax，就可以得到像這樣的新車：

```haskell
data Car = Car {company :: String, model :: String, year :: Int} deriving (Show)
```

```haskell
ghci> Car {company="Ford", model="Mustang", year=1967}
Car {company = "Ford", model = "Mustang", year = 1967}
```

這一來在造車時我們就不必關心各項的順序了。

表示三維向量之類簡單數據，`Vector = Vector Int Int Int` 就足夠明白了。但一個值構造子中若含有很多個項且不易區分，如一個人或者一輛車啥的，就應該使用 Record Syntax。

## Type parameters

值構造子可以取幾個參數產生一個新值，如 `Car` 的構造子是取三個參數返回一個 `Car`。與之相似，型別構造子可以取型別作參數，產生新的型別。這咋一聽貌似有點深奧，不過實際上並不複雜。如果你對 C++ 的模板有瞭解，就會看到很多相似的地方。我們看一個熟悉的型別，好對型別參數有個大致印象：

```haskell
data Maybe a = Nothing | Just a
```

![](img/yeti.png)

這裡的a就是個型別參數。也正因為有了它，`Maybe` 就成為了一個型別構造子。在它的值不是 `Nothing` 時，它的型別構造子可以搞出 `Maybe Int`，`Maybe String` 等等諸多型別。但只一個 `Maybe` 是不行的，因為它不是型別，而是型別構造子。要成為真正的型別，必須得把它需要的型別參數全部填滿。

所以，如果拿 `Char` 作參數交給 `Maybe`，就可以得到一個 `Maybe Char` 的型別。如，`Just 'a'` 的型別就是 `Maybe Char` 。

你可能並未察覺，在遇見 Maybe 之前我們早就接觸到型別參數了。它便是 List 型別。這裡面有點語法糖，List 型別實際上就是取一個參數來生成一個特定型別，這型別可以是 `[Int]`，`[Char]` 也可以是 `[String]`，但不會跟在 `[]` 的後面。

把玩一下 `Maybe`！

```haskell
ghci> Just "Haha"
Just "Haha"
ghci> Just 84
Just 84
ghci> :t Just "Haha"
Just "Haha" :: Maybe [Char]
ghci> :t Just 84
Just 84 :: (Num t) => Maybe t
ghci> :t Nothing
Nothing :: Maybe a
ghci> Just 10 :: Maybe Double
Just 10.0
```

型別參數很實用。有了它，我們就可以按照我們的需要構造出不同的型別。若執行 `:t Just "Haha"`，型別推導引擎就會認出它是個 `Maybe [Char]`，由於 `Just a` 裡的 `a` 是個字元串，那麼 `Maybe a` 裡的 `a` 一定也是個字元串。

![](img/meekrat.png)

注意下，`Nothing` 的型別為 `Maybe a`。它是多態的，若有函數取 `Maybe Int` 型別的參數，就一概可以傳給它一個 `Nothing`，因為 `Nothing` 中不包含任何值。`Maybe a` 型別可以有 `Maybe Int` 的行為，正如 `5` 可以是 `Int` 也可以是 `Double`。與之相似，空 List 的型別是 `[a]`，可以與一切 List 打交道。因此，我們可以 `[1,2,3]++[]`，也可以 `["ha","ha,","ha"]++[]`。

型別參數有很多好處，但前提是用對了地方才行。一般都是不關心型別裡面的內容，如 `Maybe a`。一個型別的行為若有點像是容器，那麼使用型別參數會是個不錯的選擇。我們完全可以把我們的`Car`型別從

```haskell
data Car = Car { company :: String
                 , model :: String
                 , year :: Int
                 } deriving (Show)
```

改成：

```haskell
data Car a b c = Car { company :: a
                       , model :: b
                       , year :: c
                        } deriving (Show)
```

但是，這樣我們又得到了什麼好處？回答很可能是，一無所得。因為我們只定義了處理 `Car String String Int` 型別的函數，像以前，我們還可以弄個簡單函數來描述車的屬性。

```haskell
tellCar :: Car -> String
tellCar (Car {company = c, model = m, year = y}) = "This " ++ c ++ " " ++ m ++ " was made in " ++ show y
```

```haskell
ghci> let stang = Car {company="Ford", model="Mustang", year=1967}
ghci> tellCar stang
"This Ford Mustang was made in 1967"
```

可愛的小函數！它的型別聲明得很漂亮，而且工作良好。好，如果改成 `Car a b c` 又會怎樣？

```haskell
tellCar :: (Show a) => Car String String a -> String
tellCar (Car {company = c, model = m, year = y}) = "This " ++ c ++ " " ++ m ++ " was made in " ++ show y
```

我們只能強制性地給這個函數安一個 `(Show a) => Car String String a` 的型別約束。看得出來，這要繁複得多。而唯一的好處貌似就是，我們可以使用 `Show` 型別類的 `instance` 來作 `a` 的型別。

```haskell
ghci> tellCar (Car "Ford" "Mustang" 1967)
"This Ford Mustang was made in 1967"
ghci> tellCar (Car "Ford" "Mustang" "nineteen sixty seven")
"This Ford Mustang was made in \"nineteen sixty seven\""
ghci> :t Car "Ford" "Mustang" 1967
Car "Ford" "Mustang" 1967 :: (Num t) => Car [Char] [Char] t
ghci> :t Car "Ford" "Mustang" "nineteen sixty seven"
Car "Ford" "Mustang" "nineteen sixty seven" :: Car [Char] [Char] [Char]
```

其實在現實生活中，使用 `Car String String Int` 在大多數情況下已經滿夠了。所以給 `Car` 型別加型別參數貌似並沒有什麼必要。通常我們都是都是在一個型別中包含的型別並不影響它的行為時才引入型別參數。一組什麼東西組成的 List 就是一個 List，它不關心裡面東西的型別是啥，然而總是工作良好。若取一組數字的和，我們可以在後面的函數體中明確是一組數字的 List。Maybe 與之相似，它表示可以有什麼東西可以沒有，而不必關心這東西是啥。

我們之前還遇見過一個型別參數的應用，就是 `Data.Map` 中的 `Map k v`。 `k` 表示 Map 中鍵的型別，`v` 表示值的型別。這是個好例子，Map 中型別參數的使用允許我們能夠用一個型別索引另一個型別，只要鍵的型別在 `Ord` 型別類就行。如果叫我們自己定義一個 Map 型別，可以在 `data` 聲明中加上一個型別類的約束。

```haskell
data (Ord k) => Map k v = ...
```

然而 Haskell 中有一個嚴格的約定，那就是永遠不要在 `data` 聲明中添加型別約束。為啥？嗯，因為這樣沒好處，反而得寫更多不必要的型別約束。`Map k v` 要是有 `Ord k` 的約束，那就相當於假定每個 Map 的相關函數都認為 `k` 是可排序的。若不給數據型別加約束，我們就不必給那些不關心鍵是否可排序的函數另加約束了。這類函數的一個例子就是 `toList`，它只是把一個 Map 轉換為關聯 List 罷了，型別聲明為 `toList :: Map k v -> [(k, v)]`。要是加上型別約束，就只能是 `toList :: (Ord k) =>Map k a -> [(k,v)]`，明顯沒必要嘛。

所以說，永遠不要在 `data` 聲明中加型別約束 --- 即便看起來沒問題。免得在函數聲明中寫出過多無謂的型別約束。

我們實現個表示三維向量的型別，再給它加幾個處理函數。我麼那就給它個型別參數，雖然大多數情況都是數值型，不過這一來它就支持了多種數值型別。

```haskell
data Vector a = Vector a a a deriving (Show)
vplus :: (Num t) => Vector t -> Vector t -> Vector t
(Vector i j k) `vplus` (Vector l m n) = Vector (i+l) (j+m) (k+n)
vectMult :: (Num t) => Vector t -> t -> Vector t
(Vector i j k) `vectMult` m = Vector (i*m) (j*m) (k*m)
scalarMult :: (Num t) => Vector t -> Vector t -> t
(Vector i j k) `scalarMult` (Vector l m n) = i*l + j*m + k*n
```

`vplus` 用來相加兩個向量，即將其所有對應的項相加。`scalarMult` 用來求兩個向量的標量積，`vectMult` 求一個向量和一個標量的積。這些函數可以處理 `Vector Int`，`Vector Integer`，`Vector Float` 等等型別，只要 `Vector a` 裡的這個 `a` 在 `Num` 型別類中就行。同樣，如果你看下這些函數的型別聲明就會發現，它們只能處理相同型別的向量，其中包含的數字型別必須與另一個向量一致。注意，我們並沒有在 `data` 聲明中添加 `Num` 的類約束。反正無論怎麼著都是給函數加約束。

再度重申，型別構造子和值構造子的區分是相當重要的。在聲明數據型別時，等號=左端的那個是型別構造子，右端的\(中間可能有\|分隔\)都是值構造子。拿 `Vector t t t -> Vector t t t -> t` 作函數的型別就會產生一個錯誤，因為在型別聲明中只能寫型別，而 `Vector` 的型別構造子只有個參數，它的值構造子才是有三個。我們就慢慢耍：

```haskell
ghci> Vector 3 5 8 `vplus` Vector 9 2 8
Vector 12 7 16
ghci> Vector 3 5 8 `vplus` Vector 9 2 8 `vplus` Vector 0 2 3
Vector 12 9 19
ghci> Vector 3 9 7 `vectMult` 10
Vector 30 90 70
ghci> Vector 4 9 5 `scalarMult` Vector 9.0 2.0 4.0
74.0
ghci> Vector 2 9 3 `vectMult` (Vector 4 9 5 `scalarMult` Vector 9 2 4)
Vector 148 666 222
```

## Derived instances

![](img/gob.png)

在 \[types-and-type-classes.html\#Typeclasses入門 Typeclass 101\] 那一節裡面，我們瞭解了 Typeclass 的基礎內容。裡面提到，型別類就是定義了某些行為的介面。例如，Int 型別是 `Eq` 型別類的一個 instance，`Eq` 類就定義了判定相等性的行為。Int 值可以判斷相等性，所以 Int 就是 `Eq` 型別類的成員。它的真正威力體現在作為 `Eq` 介面的函數中，即 `==` 和 `/=`。只要一個型別是 `Eq` 型別類的成員，我們就可以使用 `==` 函數來處理這一型別。這便是為何 `4==4` 和 `"foo"/="bar"` 這樣的表達式都需要作型別檢查。

我們也曾提到，人們很容易把型別類與 Java，Python，C++ 等語言的類混淆。很多人對此都倍感不解，在原先那些語言中，類就像是藍圖，我們可以根據它來創造對象、保存狀態並執行操作。而型別類更像是介面，我們不是靠它構造數據，而是給既有的數據型別描述行為。什麼東西若可以判定相等性，我們就可以讓它成為 `Eq` 型別類的 instance。什麼東西若可以比較大小，那就可以讓它成為 `Ord` 型別類的 instance。

在下一節，我們將看一下如何手工實現型別類中定義函數來構造 instance。現在呢，我們先瞭解下 Haskell 是如何自動生成這幾個型別類的 instance，`Eq`, `Ord`, `Enum`, `Bounded`, `Show`, `Read`。只要我們在構造型別時在後面加個 `deriving`\(派生\)關鍵字，Haskell 就可以自動地給我們的型別加上這些行為。

看這個數據型別：

```haskell
data Person = Person { firstName :: String
                     , lastName :: String
                     , age :: Int
                     }
```

這描述了一個人。我們先假定世界上沒有重名重姓又同齡的人存在，好，假如有兩個 record，有沒有可能是描述同一個人呢？當然可能，我麼可以判定姓名年齡的相等性，來判斷它倆是否相等。這一來，讓這個型別成為 `Eq` 的成員就很靠譜了。直接 derive 這個 instance：

```haskell
data Person = Person { firstName :: String
                     , lastName :: String
                     , age :: Int
                     } deriving (Eq)
```

在一個型別 derive 為 `Eq` 的 instance 後，就可以直接使用 `==` 或 `/=` 來判斷它們的相等性了。Haskell 會先看下這兩個值的值構造子是否一致\(這裡只是單值構造子\)，再用 `==` 來檢查其中的所有數據\(必須都是 `Eq` 的成員\)是否一致。在這裡只有 `String` 和 Int，所以是沒有問題的。測試下我們的 Eqinstance：

```haskell
ghci> let mikeD = Person {firstName = "Michael", lastName = "Diamond", age = 43}
ghci> let adRock = Person {firstName = "Adam", lastName = "Horovitz", age = 41}
ghci> let mca = Person {firstName = "Adam", lastName = "Yauch", age = 44}
ghci> mca == adRock
False
ghci> mikeD == adRock
False
ghci> mikeD == mikeD
True
ghci> mikeD == Person {firstName = "Michael", lastName = "Diamond", age = 43}
True
```

自然，`Person` 如今已經成為了 `Eq` 的成員，我們就可以將其應用於所有在型別聲明中用到 `Eq` 類約束的函數了，如 `elem`。

```haskell
ghci> let beastieBoys = [mca, adRock, mikeD]
ghci> mikeD `elem` beastieBoys
True
```

`Show` 和 `Read` 型別類處理可與字元串相互轉換的東西。同 `Eq` 相似，如果一個型別的構造子含有參數，那所有參數的型別必須都得屬於 `Show` 或 `Read` 才能讓該型別成為其 instance。就讓我們的 `Person` 也成為 `Read` 和 `Show` 的一員吧。

```haskell
data Person = Person { firstName :: String
                     , lastName :: String
                     , age :: Int
                     } deriving (Eq, Show, Read)
```

然後就可以輸出一個 `Person` 到控制台了。

```haskell
ghci> let mikeD = Person {firstName = "Michael", lastName = "Diamond", age = 43}
ghci> mikeD
Person {firstName = "Michael", lastName = "Diamond", age = 43}
ghci> "mikeD is: " ++ show mikeD
"mikeD is: Person {firstName = \"Michael\", lastName = \"Diamond\", age = 43}"
```

如果我們還沒讓 `Person` 型別作為 `Show` 的成員就嘗試輸出它，Haskell 就會向我們抱怨，說它不知道該怎麼把它表示成一個字元串。不過現在既然已經 derive 成為了 `Show` 的一個 instance，它就知道了。

`Read` 几乎就是與 `Show` 相對的型別類，`show` 是將一個值轉換成字元串，而 `read` 則是將一個字元串轉成某型別的值。還記得，使用 `read` 函數時我們必須得用型別註釋註明想要的型別，否則 Haskell 就不會知道如何轉換。

```haskell
ghci> read "Person {firstName =\"Michael\", lastName =\"Diamond\", age = 43}" :: Person
Person {firstName = "Michael", lastName = "Diamond", age = 43}
```

如果我們 `read` 的結果會在後面用到參與計算，Haskell 就可以推導出是一個 Person 的行為，不加註釋也是可以的。

```haskell
ghci> read "Person {firstName =\"Michael\", lastName =\"Diamond\", age = 43}" == mikeD
True
```

也可以 `read` 帶參數的型別，但必須填滿所有的參數。因此 `read "Just 't'" :: Maybe a` 是不可以的，`read "Just 't'" :: Maybe Char` 才對。

很容易想象 `Ord` 型別類 derive instance 的行為。首先，判斷兩個值構造子是否一致，如果是，再判斷它們的參數，前提是它們的參數都得是 `Ord` 的 instance。`Bool` 型別可以有兩種值，`False` 和 `True`。為了瞭解在比較中程序的行為，我們可以這樣想象：

```haskell
data Bool = False | True deriving (Ord)
```

由於值構造子 `False` 安排在 `True` 的前面，我們可以認為 `True` 比 `False` 大。

```haskell
ghci> True `compare` False
GT
ghci> True > False
True
ghci> True < False
False
```

在 `Maybe a` 數據型別中，值構造子 `Nothing` 在 `Just` 值構造子前面，所以一個 `Nothing` 總要比 `Just something` 的值小。即便這個 `something` 是 `-100000000` 也是如此。

```haskell
ghci> Nothing < Just 100
True
ghci> Nothing > Just (-49999)
False
ghci> Just 3 `compare` Just 2
GT
ghci> Just 100 > Just 50
True
```

不過類似 `Just (*3) > Just (*2)` 之類的程式碼是不可以的。因為 `(*3)` 和 `(*2)` 都是函數，而函數不是 `Ord` 類的成員。

作枚舉，使用數字型別就能輕易做到。不過使用 `Enum` 和 `Bounded` 型別類會更好，看下這個型別：

```haskell
data Day = Monday | Tuesday | Wednesday | Thursday | Friday | Saturday | Sunday
```

所有的值構造子都是 `nullary` 的\(也就是沒有參數\)，每個東西都有前置子和後繼子，我們可以讓它成為 `Enum` 型別類的成員。同樣，每個東西都有可能的最小值和最大值，我們也可以讓它成為 `Bounded` 型別類的成員。在這裡，我們就同時將它搞成其它可 derive型別類的 instance。再看看我們能拿它做啥：

```haskell
data Day = Monday | Tuesday | Wednesday | Thursday | Friday | Saturday | Sunday
           deriving (Eq, Ord, Show, Read, Bounded, Enum)
```

由於它是 `Show` 和 `Read` 型別類的成員，我們可以將這個型別的值與字元串相互轉換。

```haskell
ghci> Wednesday
Wednesday
ghci> show Wednesday
"Wednesday"
ghci> read "Saturday" :: Day
Saturday
```

由於它是 `Eq` 與 `Ord` 的成員，因此我們可以拿 `Day` 作比較。

```haskell
ghci> Saturday == Sunday
False
ghci> Saturday == Saturday
True
ghci> Saturday > Friday
True
ghci> Monday `compare` Wednesday
LT
```

它也是 `Bounded` 的成員，因此有最早和最晚的一天。

```haskell
ghci> minBound :: Day
Monday
ghci> maxBound :: Day
Sunday
```

它也是 `Enum` 的 instance，可以得到前一天和後一天，並且可以對此使用 List 的區間。

```haskell
ghci> succ Monday
Tuesday
ghci> pred Saturday
Friday
ghci> [Thursday .. Sunday]
[Thursday,Friday,Saturday,Sunday]
ghci> [minBound .. maxBound] :: [Day]
[Monday,Tuesday,Wednesday,Thursday,Friday,Saturday,Sunday]
```

那是相當的棒。

## Type synonyms

在前面我們提到在寫型別名的時候，`[Char]` 和 `String` 等價，可以互換。這就是由型別別名實現的。型別別名實際上什麼也沒做，只是給型別提供了不同的名字，讓我們的程式碼更容易理解。這就是 `[Char]` 的別名 `String` 的由來。

```haskell
type String = [Char]
```

我們已經介紹過了 `type` 關鍵字，這個關鍵字有一定誤導性，它並不是用來創造新類（這是 `data` 關鍵字做的事情），而是給一個既有型別提供一個別名。

如果我們隨便搞個函數 `toUpperString` 或其他什麼名字，將一個字元串變成大寫，可以用這樣的型別聲明 `toUpperString :: [Char] -> [Char]`， 也可以這樣 `toUpperString :: String -> String`，二者在本質上是完全相同的。後者要更易讀些。

在前面 `Data.Map` 那部分，我們用了一個關聯 `List` 來表示 `phoneBook`，之後才改成的 Map。我們已經發現了，一個關聯 List 就是一組鍵值對組成的 List。再看下我們 `phoneBook` 的樣子：

```haskell
phoneBook :: [(String,String)]
phoneBook =
    [("betty","555-2938")
    ,("bonnie","452-2928")
    ,("patsy","493-2928")
    ,("lucille","205-2928")
    ,("wendy","939-8282")
    ,("penny","853-2492")
    ]
```

可以看出，`phoneBook` 的型別就是 `[(String,String)]`，這表示一個關聯 List 僅是 String 到 String 的映射關係。我們就弄個型別別名，好讓它型別聲明中能夠表達更多資訊。

```haskell
type PhoneBook = [(String,String)]
```

現在我們 `phoneBook` 的型別聲明就可以是 `phoneBook :: PhoneBook` 了。再給字元串加上別名：

```haskell
type PhoneNumber = String
type Name = String
type PhoneBook = [(Name,PhoneNumber)]
```

Haskell 程序員給 String 加別名是為了讓函數中字元串的表達方式及用途更加明確。

好的，我們實現了一個函數，它可以取一名字和號碼檢查它是否存在於電話本。現在可以給它加一個相當好看明了的型別聲明：

```haskell
inPhoneBook :: Name -> PhoneNumber -> PhoneBook -> Bool
inPhoneBook name pnumber pbook = (name,pnumber) `elem` pbook
```

![](img/chicken.png)

如果不用型別別名，我們函數的型別聲明就只能是 `String -> String -> [(String ,String)] -> Bool` 了。在這裡使用型別別名是為了讓型別聲明更加易讀，但你也不必拘泥于它。引入型別別名的動機既非單純表示我們函數中的既有型別，也不是為了替換掉那些重複率高的長名字型別\(如 `[(String,String)]`\)，而是為了讓型別對事物的描述更加明確。

型別別名也是可以有參數的，如果你想搞個型別來表示關聯 List，但依然要它保持通用，好讓它可以使用任意型別作 `key` 和 `value`，我們可以這樣：

```haskell
type AssocList k v = [(k,v)]
```

好的，現在一個從關聯 List 中按鍵索值的函數型別可以定義為 `(Eq k) => k -> AssocList k v -> Maybe v. AssocList i`。`AssocList` 是個取兩個型別做參數生成一個具體型別的型別構造子，如 `Assoc Int String` 等等。

```text
*Fronzie 說*：Hey！當我提到具體型別，那我就是說它是完全呼叫的，就像 ``Map Int String``。要不就是多態函數中的 ``[a]`` 或 ``(Ord a) => Maybe a`` 之類。有時我和孩子們會說 "Maybe 型別"，但我們的意思並不是按字面來，傻瓜都知道 ``Maybe`` 是型別構造子嘛。只要用一個明確的型別呼叫 ``Maybe``，如 ``Maybe String`` 可得一個具體型別。你知道，只有具體型別才可以儲存值。
```

我們可以用不全呼叫來得到新的函數，同樣也可以使用不全呼叫得到新的型別構造子。同函數一樣，用不全的型別參數呼叫型別構造子就可以得到一個不全呼叫的型別構造子，如果我們要一個表示從整數到某東西間映射關係的型別，我們可以這樣：

```haskell
type IntMap v = Map Int v
```

也可以這樣：

```haskell
type IntMap = Map Int
```

無論怎樣，`IntMap` 的型別構造子都是取一個參數，而它就是這整數指向的型別。

Oh yeah，如果要你去實現它，很可能會用個 `qualified import` 來導入 `Data.Map`。這時，型別構造子前面必須得加上模組名。所以應該寫個 `type IntMap = Map.Map Int`

你得保證真正弄明白了型別構造子和值構造子的區別。我們有了個叫 `IntMap` 或者 `AssocList` 的別名並不意味着我們可以執行類似 `AssocList [(1,2),(4,5),(7,9)]` 的程式碼，而是可以用不同的名字來表示原先的 List，就像 `[(1,2),(4,5),(7,9)] :: AssocList Int Int` 讓它裡面的型別都是 Int。而像處理普通的 Tuple 構成的那種 List 處理它也是可以的。型別別名\(型別依然不變\)，只可以在 Haskell 的型別部分中使用，像定義新型別或型別聲明或型別註釋中跟在::後面的部分。

另一個很酷的二參型別就是 `Either a b` 了，它大約是這樣定義的：

```haskell
data Either a b = Left a | Right b deriving (Eq, Ord, Read, Show)
```

它有兩個值構造子。如果用了 `Left`，那它內容的型別就是 `a`；用了 `Right`，那它內容的型別就是 `b`。我們可以用它來將可能是兩種型別的值封裝起來，從裡面取值時就同時提供 `Left` 和 `Right` 的模式匹配。

```haskell
ghci> Right 20
Right 20
ghci> Left "w00t"
Left "w00t"
ghci> :t Right 'a'
Right 'a' :: Either a Char
ghci> :t Left True
Left True :: Either Bool b
```

到現在為止，`Maybe` 是最常見的表示可能失敗的計算的型別了。但有時 `Maybe` 也並不是十分的好用，因為 `Nothing` 中包含的信息還是太少。要是我們不關心函數失敗的原因，它還是不錯的。就像 `Data.Map` 的 `lookup` 只有在搜尋的項不在 Map 時才會失敗，對此我們一清二楚。但我們若想知道函數失敗的原因，那還得使用 `Either a b`，用 `a` 來表示可能的錯誤的型別，用 `b` 來表示一個成功運算的型別。從現在開始，錯誤一律用 `Left` 值構造子，而結果一律用 `Right`。

一個例子：有個學校提供了不少壁櫥，好給學生們地方放他們的 Gun'N'Rose 海報。每個壁櫥都有個密碼，哪個學生想用個壁櫥，就告訴管理員壁櫥的號碼，管理員就會告訴他壁櫥的密碼。但如果這個壁櫥已經讓別人用了，管理員就不能告訴他密碼了，得換一個壁櫥。我們就用 `Data.Map` 的一個 Map 來表示這些壁櫥，把一個號碼映射到一個表示壁櫥占用情況及密碼的 Tuple 裡。

```haskell
import qualified Data.Map as Map

data LockerState = Taken | Free deriving (Show, Eq)

type Code = String

type LockerMap = Map.Map Int (LockerState, Code)
```

很簡單，我們引入了一個新的型別來表示壁櫥的占用情況。併為壁櫥密碼及按號碼找壁櫥的 Map 分別設置了一個別名。好，現在我們實現這個按號碼找壁櫥的函數，就用 `Either String Code` 型別表示我們的結果，因為 `lookup` 可能會以兩種原因失敗。櫥子已經讓別人用了或者壓根就沒有這個櫥子。如果 `lookup` 失敗，就用字元串表明失敗的原因。

```haskell
lockerLookup :: Int -> LockerMap -> Either String Code
lockerLookup lockerNumber map =
    case Map.lookup lockerNumber map of
        Nothing -> Left $ "Locker number " ++ show lockerNumber ++ " doesn't exist!"
        Just (state, code) -> if state /= Taken
                                then Right code
                                else Left $ "Locker " ++ show lockerNumber ++ " is already taken!"
```

我們在這裡個 Map 中執行一次普通的 `lookup`，如果得到一個 `Nothing`，就返回一個 `Left String` 的值，告訴他壓根就沒這個號碼的櫥子。如果找到了，就再檢查下，看這櫥子是不是已經讓別人用了，如果是，就返回個 `Left String` 說它已經讓別人用了。否則就返回個 `Right Code` 的值，通過它來告訴學生壁櫥的密碼。它實際上就是個 `Right String`，我們引入了個型別別名讓它這型別聲明更好看。

如下是個 Map 的例子：

```haskell
lockers :: LockerMap
lockers = Map.fromList
    [(100,(Taken,"ZD39I"))
    ,(101,(Free,"JAH3I"))
    ,(103,(Free,"IQSA9"))
    ,(105,(Free,"QOTSA"))
    ,(109,(Taken,"893JJ"))
    ,(110,(Taken,"99292"))
    ]
```

現在從裡面 `lookup` 某個櫥子號..

```haskell
ghci> lockerLookup 101 lockers
Right "JAH3I"
ghci> lockerLookup 100 lockers
Left "Locker 100 is already taken!"
ghci> lockerLookup 102 lockers
Left "Locker number 102 doesn't exist!"
ghci> lockerLookup 110 lockers
Left "Locker 110 is already taken!"
ghci> lockerLookup 105 lockers
Right "QOTSA"
```

我們完全可以用 `Maybe a` 來表示它的結果，但這樣一來我們就對得不到密碼的原因不得而知了。而在這裡，我們的新型別可以告訴我們失敗的原因。

## Recursive data structures \(遞迴地定義資料結構\)

如我們先前看到的，一個 algebraic data type 的構造子可以有好幾個 field，其中每個 field 都必須有具體的型態。有了那個概念，我們能定義一個型態，其中他的構造子的 field 的型態是他自己。這樣我們可以遞迴地定義下去，某個型態的值便可能包含同樣型態的值，進一步下去他還可以再包含同樣型態的值。

考慮一下 List: `[5]`。他其實是 `5:[]` 的語法糖。在 `:` 的左邊是一個普通值，而在右邊是一串 List。只是在這個案例中是空的 List。再考慮 `[4,5]`。他可以看作 `4:(5:[])`。看看第一個 `:`，我們看到他也有一個元素在左邊，一串 List `5:[]` 在右邊。同樣的道理 `3:(4:(5:6:[]))` 也是這樣。

我們可以說一個 List 的定義是要碼是空的 List 或是一個元素，後面用 `:` 接了另一串 List。

我們用 algebraic data type 來實作我們自己的 List！

```haskell
data List a = Empty | Cons a (List a) deriving (Show, Read, Eq, Ord)
```

這讀起來好像我們前一段提及的定義。他要碼是空的 List，或是一個元素跟一串 List 的結合。如果你被搞混了，看看用 record syntax 定義的可能比較清楚。

```haskell
data List a = Empty | Cons { listHead :: a, listTail :: List a} deriving (Show, Read, Eq, Ord)
```

你可能也對這邊的 `Cons` 構造子不太清楚。`cons` 其實就是指 `:`。對 List 而言，`:` 其實是一個構造子，他接受一個值跟另一串 List 來構造一個 List。現在我們可以使用我們新定義的 List 型態。換句話說，他有兩個 field，其中一個 field 具有型態 `a`，另一個有型態 `[a]`。

```haskell
ghci> Empty
Empty
ghci> 5 `Cons` Empty
Cons 5 Empty
ghci> 4 `Cons` (5 `Cons` Empty)
Cons 4 (Cons 5 Empty)
ghci> 3 `Cons` (4 `Cons` (5 `Cons` Empty))
Cons 3 (Cons 4 (Cons 5 Empty))
```

我們用中綴的方式呼叫 `Cons` 構造子，這樣你可以很清楚地看到他就是 `:`。`Empty` 代表 `[]`，而 ``4 `Cons` (5 `Cons` Empty)`` 就是 `4:(5:[])`。

我們可以只用特殊字元來定義函數，這樣他們就會自動具有中綴的性質。我們也能同樣的手法套用在構造子上，畢竟他們不過是回傳型態的函數而已。

```haskell
infixr 5 :-:
data List a = Empty | a :-: (List a) deriving (Show, Read, Eq, Ord)
```

首先我們留意新的語法結構：fixity 宣告。當我們定義函數成 operator，我們能同時指定 fixity \(但並不是必須的\)。fixity 指定了他應該是 left-associative 或是 right-associative，還有他的優先順序。例如說，`*` 的 fixity 是 `infixl 7 *`，而 `+` 的 fixity 是 `infixl 6`。代表他們都是 left-associative。`(4 * 3 * 2)` 等於 `((4 * 3) * 2)`。但 `*` 擁有比 `+` 更高的優先順序。所以 `5 * 4 + 3` 會是 `(5 * 4) + 3`。

這樣我們就可以寫成 `a :-: (List a)` 而不是 `Cons a (List a)`：

```haskell
ghci> 3 :-: 4 :-: 5 :-: Empty
(:-:) 3 ((:-:) 4 ((:-:) 5 Empty))
ghci> let a = 3 :-: 4 :-: 5 :-: Empty
ghci> 100 :-: a
(:-:) 100 ((:-:) 3 ((:-:) 4 ((:-:) 5 Empty)))
```

Haskell 在宣告 `deriving Show` 的時候，他會仍視構造子為前綴函數，因此必須要用括號括起來。

我們在來寫個函數來把兩個 List 連起來。一般 `++` 在操作普通 List 的時候是這樣的：

```haskell
infixr 5  ++
(++) :: [a] -> [a] -> [a]
[]     ++ ys = ys
(x:xs) ++ ys = x : (xs ++ ys)
```

我們把他偷過來用在我們的 List 上，把函數命名成 `.++`：

```haskell
infixr 5  .++
(.++) :: List a -> List a -> List a
Empty .++ ys = ys
(x :-: xs) .++ ys = x :-: (xs .++ ys)
```

來看看他如何運作：

```haskell
ghci> let a = 3 :-: 4 :-: 5 :-: Empty
ghci> let b = 6 :-: 7 :-: Empty
ghci> a .++ b
(:-:) 3 ((:-:) 4 ((:-:) 5 ((:-:) 6 ((:-:) 7 Empty))))
```

如果我們想要的話，我們可以定義其他操作我們list的函數。

注意到我們是如何利用 `(x :-: xs)` 做模式匹配的。他運作的原理實際上就是利用到構造子。我們可以利用 `:-:` 做模式匹配原因就是他是構造子，同樣的 `:` 也是構造子所以可以用他做匹配。`[]` 也是同樣道理。由於模式匹配是用構造子來作的，所以我們才能對像 `8`, `'a'` 之類的做模式匹配。他們是數值與字元的構造子。

接下來我們要實作二元搜尋樹 \(binary search tree\)。如果你對二元搜尋樹不太清楚，我們來快速地解釋一遍。他的結構是每個節點指向兩個其他節點，一個在左邊一個在右邊。在左邊節點的元素會比這個節點的元素要小。在右邊的話則比較大。每個節點最多可以有兩棵子樹。而我們知道譬如說一棵包含 5 的節點的左子樹，裡面所有的元素都會小於 5。而節點的右子樹裡面的元素都會大於 5。如果我們想找找看 8是不是在我們的樹裡面，我們就從 5 那個節點找起，由於 8 比 5 要大，很自然地就會往右搜尋。接著我們走到 7，又由於 8 比 7 要大，所以我們再往右走。我們在三步就找到了我們要的元素。如果這不是棵樹而是 List 的話，那就會需要花到七步才能找到 8。

`Data.Set` 跟 `Data.Map` 中的 `set` 和 Map 都是用樹來實現的，只是他們是用平衡二元搜尋樹而不是隨意的二元搜尋樹。不過這邊我們就只先寫一棵普通的二元搜尋樹就好了。

這邊我們來定義一棵樹的結構：他不是一棵空的樹就是帶有值並含有兩棵子樹。聽起來非常符合 algebraic data type 的結構！

```haskell
data Tree a = EmptyTree | Node a (Tree a) (Tree a) deriving (Show, Read, Eq)
```

我們不太想手動來建棵二元搜尋樹，所以我們要來寫一個函數，他接受一棵樹還有一個元素，把這個元素安插到這棵二元搜尋樹中。當拿這個元素跟樹的節點比較結果比較小的話，我們就往左走，如果比較大，就往右走。重複這個動作直到我們走到一棵空的樹。一旦碰到空的樹的話，我們就把元素插入節點。

在 C 語言中，我們是用修改指標的方式來達成這件事。但在 Haskell 中，我們沒辦法修改我們的樹。所以我們在決定要往左或往右走的時候就做一棵新的子樹，走到最後要安插節點的時候也是做一棵新的樹。因此我們插入函數的型態會是 `a -> Tree a -> Tree a`。他接受一個元素跟一棵樹，並回傳一棵包含了新元素的新的樹。這看起來很沒效率的樣子，但別擔心，惰性的特性可以讓我們不用擔心這個。

來看下列兩個函數。第一個做了一個單節點的樹，而第二個插入一個元素到一棵樹中。

```haskell
singleton :: a -> Tree a
singleton x = Node x EmptyTree EmptyTree

treeInsert :: (Ord a) => a -> Tree a -> Tree a
treeInsert x EmptyTree = singleton x
treeInsert x (Node a left right)
      | x == a = Node x left right
      | x < a  = Node a (treeInsert x left) right
      | x > a  = Node a left (treeInsert x right)
```

`singleton` 函數只是一個做一個含有兩棵空子樹的節點的函數的別名。在插入的操作中，我們先為終端條件定義了一個模式匹配。如果我們走到了一棵空的子樹，這表示我們到達了我們想要的地方，我們便建造一棵空的單元素的樹來放在那個位置。如果我們還沒走到一棵空的樹來插入我們的元素。那就必須要做一些檢查來往下走。如果我們要安插的元素跟 root 所含有的元素相等，那就直接回傳這棵樹。如果安插的元素比較小，就回傳一棵新的樹。這棵樹的 root 跟原來的相同，右子樹也相同，只差在我們要安插新的元素到左子樹中。如果安插的元素反而比較大，那整個過程就相反。

接下來，我們要寫一個函數來檢查某個元素是否已經在這棵樹中。首先我們定義終端條件。如果我們已經走到一棵空的樹，那這個元素一定不在這棵樹中。這跟我們搜尋 List 的情形是一致的。如果我們要在空的 List 中搜尋某一元素，那就代表他不在這個 List 裡面。假設我們現在搜尋一棵非空的樹，而且 root 中的元素剛好就是我們要的，那就找到了。那如果不是呢？我們就要利用在 root 節點左邊的元素都比 root 小的這個性質。如果我們的元素比 root 小，那就往左子樹中找。如果比較大，那就往右子樹中找。

```haskell
treeElem :: (Ord a) => a -> Tree a -> Bool
treeElem x EmptyTree = False
treeElem x (Node a left right)
    | x == a = True
    | x < a  = treeElem x left
    | x > a  = treeElem x right
```

我們要作的就是把之前段落所描述的事轉換成程式碼。首先我們不想手動一個個來創造一棵樹。我們想用一個 `fold` 來從一個 List 創造一棵樹。要知道走遍一個 List 並回傳某種值的操作都可以用 `fold` 來實現。我們先從一棵空的樹開始，然後從右邊走過 List的 每一個元素，一個一個丟到樹裡面。

```haskell
ghci> let nums = [8,6,4,1,7,3,5]
ghci> let numsTree = foldr treeInsert EmptyTree nums
ghci> numsTree
Node 5 (Node 3 (Node 1 EmptyTree EmptyTree) (Node 4 EmptyTree EmptyTree)) (Node 7 (Node 6 EmptyTree EmptyTree) (Node 8 EmptyTree EmptyTree))
```

在 `foldr` 中，`treeInsert` 是做 folding 操作的函數，而 `EmptyTree` 是起始的 accumulator，`nums` 則是要被走遍的 List。

當我們想把我們的樹印出來的時候，印出來的形式會不太容易讀。但如果我們能有結構地印出來呢？我們知道 root 是 5，他有兩棵子樹，其中一個的 root 是 3 另一個則是 7。

```haskell
ghci> 8 `treeElem` numsTree
True
ghci> 100 `treeElem` numsTree
False
ghci> 1 `treeElem` numsTree
True
ghci> 10 `treeElem` numsTree
False
```

檢查元素是否屬於某棵樹的函數現在能正常運作了！

你可以看到 algebraic data structures 是非常有力的概念。我們可以使用這個結構來構造出布林值，週一到週五的概念，甚至還有二元樹。

## Typeclasses 的第二堂課

到目前為止我們學到了一些 Haskell 中的標準 typeclass，也學到了某些已經定義為他們 instance 的型別。我們知道如何讓我們自己定義的型別自動被 Haskell 所推導成標準 typeclass 的 instance。在這個章節中，我們會學到如何構造我們自己的 typeclass，並且如何構造這些 typeclass 的 type instance。

來快速複習一下什麼是 typeclass: typeclass 就像是 interface。一個 typeclass 定義了一些行為\(像是比較相不相等，比較大小順序，能否窮舉\)而我們會把希望滿足這些性質的型別定義成這些 typeclass 的 instance。typeclass 的行為是由定義的函數來描述。並寫出對應的實作。當我們把一個型別定義成某個 typeclass 的 instance，就表示我們可以對那個型別使用 typeclass 中定義的函數。

Typeclass 跟 Java 或 Python 中的 class 一點關係也沒有。這個概念讓很多人混淆，所以我希望你先忘掉所有在命令式語言中學到有關 class 的所有東西。

例如說，`Eq` 這個 typeclass 是描述可以比較相等的事物。他定義了 `==` 跟 `/=` 兩個函數。如果我們有一個型別 `Car`，而且對他們做相等比較是有意義的，那把 `Car` 作成是 `Eq` 的一個 instance 是非常合理的。

這邊來看看在 `Prelude` 之中 `Eq` 是怎麼被定義的。

```haskell
class Eq a where
    (==) :: a -> a -> Bool
    (/=) :: a -> a -> Bool
    x == y = not (x /= y)
    x /= y = not (x == y)
```

我們在這邊看到了一些奇怪的語法跟關鍵字。別擔心，你一下子就會瞭解他們的。首先，我們看到 `class Eq a where`，那代表我們定義了一個新的 typeclass 叫做 `Eq`。`a` 是一個型別變數，他代表 `a` 是任何我們在定義 instance 時的型別。他不一定要叫做 `a`。他也不一定非要一個字母不可，只要他是小寫就好。然後我們又定義了幾個函數。我們並不一定要實作函數的本體，不過必須要寫出函數的型別宣告。

如果我們寫成 `class Eq equatable where` 還有 `(==) :: equatable -> equatable -> Bool` 這樣的形式，對一些人可能比較容易理解。

總之我們實作了 `Eq` 中需要定義的函數本體，只是我們定義他的方式是用交互遞迴的形式。我們描述兩個 `Eq` 的 instance 要相等，那他們就不能不一樣，而他們如果不一樣，那他們就是不相等。我們其實不必這樣寫，但很快你會看到這其實是有用的。

如果我們說 `class Eq a where` 然後定義 `(==) :: a -> a -> Bool`，那我們之後檢查函數的型別時會發現他的型別是 `(Eq a) => a -> a -> Bool`。

當我們有了 class 以後，可以用來做些什麼呢？說實話，不多。不過一旦我們為它寫一些 instance，就會有些好功能。來看看下面這個型別：

```haskell
data TrafficLight = Red | Yellow | Green
```

這裡定義了紅綠燈的狀態。請注意這個型別並不是任何 class 的 instance，雖然可以透過 derive 讓它成為 `Eq` 或 `Show` 的 instance，但我們打算手工打造。下面展示了如何讓一個型別成為 `Eq` 的 instance：

```haskell
instance Eq TrafficLight where
    Red == Red = True
    Green == Green = True
    Yellow == Yellow = True
    _ == _ = False
```

我們使用了 `instance` 這個關鍵字。class 是用來定義新的 typeclass，而 instance 是用來說明我們要定義某個 typeclass 的 instance。當我們要定義 `Eq`，我們會寫 `class Eq a where`，其中 `a` 代表任何型態。我們可以從 instance 的寫法：`instance Eq TrafficLight where` 看出來。我們會把 `a` 換成實際的型別。

由於 `==` 是用 `/=` 來定義的，同樣的 `/=` 也是用 `==` 來定義。所以我們只需要在 instance 定義中複寫其中一個就好了。我們這樣叫做定義了一個 minimal complete definition。這是說能讓型別符合 class 行為所最小需要實作的函數數量。而 `Eq` 的 minimal complete definition 需要 `==` 或 `/=` 其中一個。而如果 `Eq` 是這樣定義的：

```haskell
class Eq a where
    (==) :: a -> a -> Bool
    (/=) :: a -> a -> Bool
```

當我們定義 instance 的時候必須要兩個函數都實作，因為 Haskell 並不知道這兩個函數是怎麼關聯在一起的。所以 minimal complete definition 在這邊是 `==` 跟 `/=`。

你可以看到我們是用模式匹配來實作 `==`。由於不相等的情況比較多，所以我們只寫出相等的，最後再用一個 `case` 接住說你不在前面相等的 `case` 的話，那就是不相等。

我們再來寫 `Show` 的 instance。要滿足 `Show` 的 minimal complete definition，我們必須實作 `show` 函數，他接受一個值並把他轉成字串。

```haskell
instance Show TrafficLight where
    show Red = "Red light"
    show Yellow = "Yellow light"
    show Green = "Green light"
```

再一次地，我們用模式匹配來完成我們的任務。我們來看看他是如何運作的。

```haskell
ghci> Red == Red
True
ghci> Red == Yellow
False
ghci> Red `elem` [Red, Yellow, Green]
True
ghci> [Red, Yellow, Green]
[Red light,Yellow light,Green light]
```

如果我們用 `derive` 來自動產生 `Eq` 的話，效果是一樣的。不過用 `derive` 來產生 `show` 的話，他會把值構造子轉換成字串。但我們這邊要的不太一樣，我們希望印出像 `"Red light"` 這樣的字串，所以我們就必須手動來寫出 instance。

你也可以把 typeclass 定義成其他 typeclass 的 subclass。像是 `Num` 的 class 宣告就有點冗長，但我們先看個雛型。

```haskell
class (Eq a) => Num a where
   ...
```

正如我們先前提到的，我們可以在很多地方加上 class constraints。這不過就是在 `class Num a where` 中的 `a` 上，加上他必須要是 `Eq` 的 instance 的限制。這基本上就是在說我們在定義一個型別為 `Num` 之前，必須先為他定義 `Eq` 的 instance。在某個型別可以被視作 `Number` 之前，必須先能被比較相不相等其實是蠻合理的。這就是 subclass 在做的事：幫 class declaration 加上限制。也就是說當我們定義 typeclass 中的函數本體時，我們可以預設 `a` 是屬於 `Eq`，因此能使用 `==`。

但像是 `Maybe` 或是 List 是如何被定義成 typeclass 的 instance 呢？`Maybe` 的特別之處在於他跟 `TrafficLight` 不一樣，他不是一個具體的型別。他是一個型別構造子，接受一個型別參數（像是 `Char` 之類的）而構造出一個具體的型別（像是 `Maybe Char` ）。讓我們再回顧一下 `Eq` 這個 typeclass：

```haskell
class Eq a where
    (==) :: a -> a -> Bool
    (/=) :: a -> a -> Bool
    x == y = not (x /= y)
    x /= y = not (x == y)
```

從型別宣告來看，可以看到 `a` 必須是一個具體型別，因為所有在函數中的型別都必須是具體型別。\(你沒辦法寫一個函數，他的型別是 `a -> Maybe`，但你可以寫一個函數，他的型別是 `a -> Maybe a`，或是 `Maybe Int -> Maybe String`\) 這就是為什麼我們不能寫成像這樣：

```haskell
instance Eq Maybe where
    ...
```

```haskell
instance Eq (Maybe m) where
    Just x == Just y = x == y
    Nothing == Nothing = True
    _ == _ = False
```

這就好像在說我們要把 `Maybe something` 這種東西全部都做成 `Eq` 的 instance。我們的確可以寫成 `(Maybe something)`，但我們通常是只用一個字母，這樣比較像是 Haskell 的風格。`(Maybe m)` 這邊則取代了 `a` 在 `class Eq a where` 的位置。儘管 `Maybe` 不是一個具體的型別。`Maybe m` 卻是。指定一個型別參數（在這邊是小寫的 `m`），我們說我們想要所有像是 `Maybe m` 的都成為 `Eq` 的 instance。

不過這仍然有一個問題。你能看出來嗎？ 我們用 `==` 來比較 `Maybe` 包含的東西，但我們並沒有任何保證說 `Maybe` 裝的東西可以是 `Eq`。這就是為什麼我們需要修改我們的 instance 定義：

```haskell
instance (Eq m) => Eq (Maybe m) where
    Just x == Just y = x == y
    Nothing == Nothing = True
    _ == _ = False
```

這邊我們必須要加上限制。在這個 instance 的宣告中，我們希望所有 `Maybe m` 形式的型別都屬於 `Eq`，但只有當 `m` 也屬於 `Eq` 的時候。這也是 Haskell 在 derive 的時候做的事。

在大部份情形下，在 typeclass 宣告中的 class constraints 都是要讓一個 typeclass 成為另一個 typeclass 的 subclass。而在 instance 宣告中的 class constraint 則是要表達型別的要求限制。舉裡來說，我們要求 `Maybe` 的內容物也要屬於 `Eq`。

當定義 instance 的時候，如果你需要提供具體型別（像是在 `a -> a -> Bool` 中的 `a`），那你必須要加上括號跟型別參數來構造一個具體型別。

要知道你在定義 instance 的時候，型別參數會被取代。`class Eq a where` 中的 `a` 會被取代成真實的型別。所以試著想像把型別放進型別宣告中。`(==) :: Maybe -> Maybe -> Bool` 並非合法。但 `(==) :: (Eq m) => Maybe m -> Maybe m -> Bool` 則是。這是不論我們要定義什麼，通用的型別宣告都是 `(==) :: (Eq a) => a -> a -> Bool`

還有一件事要確認。如果你想看看一個 typeclass 有定義哪些 instance。可以在 ghci 中輸入 `:info YourTypeClass`。所以輸入 `:info Num` 會告訴你這個 typeclass 定義了哪些函數，還有哪些型別屬於這個 typeclass。`:info` 也可以查詢型別跟型別構造子的資訊。如果你輸入 `:info Maybe`。他會顯示 `Maybe` 所屬的所有 typeclass。`:info` 也能告訴你函數的型別宣告。

## yes-no typeclass

在 Javascript 或是其他弱型別的程式語言，你能在 if expression 中擺上任何東西。舉例來說，你可以做像下列的事： `if (0) alert("YEAH!") else alert("NO!")`, `if ("") alert ("YEAH!") else alert("NO!")`, `if (false) alert("YEAH") else alert("NO!)` 等等， 而上述所有的片段執行後都會跳出 `NO!`。如果你寫 `if ("WHAT") alert ("YEAH") else alert("NO!")`，他會跳出 `YEAH!`，因為 Javascript 認為非空字串會是 true。

儘管使用 `Bool` 來表達布林的語意是比較好的作法。為了有趣起見，我們來試試看模仿 Javascript 的行為。我們先從 typeclass 宣告開始看：

```haskell
class YesNo a where
    yesno :: a -> Bool
```

`YesNo` typeclass 定義了一個函數。這個函數接受一個可以判斷為真否的型別的值。而從我們寫 `a` 的位置，可以看出來 `a` 必須是一個具體型別。

接下來我們來定義一些 instance。對於數字，我們會假設任何非零的數字都會被當作 `true`，而 0 則當作 `false`。

```haskell
instance YesNo Int where
    yesno 0 = False
    yesno _ = True
```

空的 List \(包含字串\)代表 `false`，而非空的 List 則代表 `true`。

```haskell
instance YesNo [a] where
    yesno [] = False
    yesno _ = True
```

留意到我們加了一個型別參數 `a` 來讓整個 List 是一個具體型別，不過我們並沒有對包涵在 List 中的元素的型別做任何額外假設。我們還剩下 `Bool` 可以被作為真假值，要定義他們也很容易：

```haskell
instance YesNo Bool where
    yesno = id
```

你說 `id` 是什麼？他不過是標準函式庫中的一個函數，他接受一個參數並回傳相同的東西。

我們也讓 `Maybe a` 成為 `YesNo` 的 instance。

```haskell
instance YesNo (Maybe a) where
    yesno (Just _) = True
    yesno Nothing = False
```

由於我們不必對 `Maybe` 的內容做任何假設，因此並不需要 class constraint。我們只要定義遇到 `Just` 包裝過的值就代表 true，而 `Nothing` 則代表 false。這裡還是得寫出 `(Maybe a)` 而不是只有 `Maybe`，畢竟 `Maybe -> Bool` 的函式並不存在（因為 `Maybe` 並不是具體型別），而 `Maybe a -> Bool` 看起來就合理多了。現在有了這個定義，`Maybe something` 型式的型別都屬於 `YesNo` 了，不論 `something` 是什麼。

之前我們定義了 `Tree a`，那代表一個二元搜尋樹。我們可以說一棵空的樹是 `false`，而非空的樹則是 `true`。

```haskell
instance YesNo (Tree a) where
    yesno EmptyTree = False
    yesno _ = True
```

而一個紅綠燈可以代表 yes or no 嗎？當然可以。如果他是紅燈，那你就會停下來，如果他是綠燈，那你就能走。但如果是黃燈呢？只能說我通常會闖黃燈。

```haskell
instance YesNo TrafficLight where
    yesno Red = False
    yesno _ = True
```

現在我們定義了許多 instance，來試著跑跑看！

```haskell
ghci> yesno $ length []
False
ghci> yesno "haha"
True
ghci> yesno ""
False
ghci> yesno $ Just 0
True
ghci> yesno True
True
ghci> yesno EmptyTree
False
ghci> yesno []
False
ghci> yesno [0,0,0]
True
ghci> :t yesno
yesno :: (YesNo a) => a -> Bool
```

很好，統統是我們預期的結果。我們來寫一個函數來模仿 if statement 的行為，但他是運作在 `YesNo` 的型別上。

```haskell
yesnoIf :: (YesNo y) => y -> a -> a -> a
yesnoIf yesnoVal yesResult noResult =
    if yesno yesnoVal then yesResult else noResult
```

很直覺吧！他接受一個 yes or no 的值還有兩個部份，如果值是代表 "yes"，那第一個部份就會被執行，而如果值是 "no"，那第二個部份就會執行。

```haskell
ghci> yesnoIf [] "YEAH!" "NO!"
"NO!"
ghci> yesnoIf [2,3,4] "YEAH!" "NO!"
"YEAH!"
ghci> yesnoIf True "YEAH!" "NO!"
"YEAH!"
ghci> yesnoIf (Just 500) "YEAH!" "NO!"
"YEAH!"
ghci> yesnoIf Nothing "YEAH!" "NO!"
"NO!"
```

## Functor typeclass

到目前為止我們看過了許多在標準函式庫中的 typeclass。我們操作過 `Ord`，代表可以被排序的東西。我們也操作過 `Eq`，代表可以被比較相等性的事物。也看過 `Show`，代表可以被印成字串來表示的東西。至於 `Read` 則是我們可以把字串轉換成型別的動作。不過現在我們要來看一下 `Functor` 這個 typeclass，基本上就代表可以被 map over 的事物。聽到這個詞你可能會聯想到 List，因為 map over list 在 Haskell 中是很常見的操作。你沒想錯，List 的確是屬於 `Functor` 這個 typeclass。

來看看他的實作會是瞭解 `Functor` 的最佳方式：

```haskell
class Functor f where
    fmap :: (a -> b) -> f a -> f b
```

我們看到他定義了一個函數 `fmap`，而且並沒有提供一個預設的實作。`fmap` 的型別蠻有趣的。到目前為止的我們看過的 typeclass 中的型別變數都是具體型別。就像是 `(==) :: (Eq a) => a -> a -> Bool` 中的 `a` 一樣。但現在碰到的 `f` 並不是一個具體型別（一個像是 `Int`, `Bool` 或 `Maybe String`的型別），而是接受一個型別參數的型別構造子。如果要快速回顧的話可以看一下 `Maybe Int` 是一個具體型別，而 `Maybe` 是一個型別構造子，可接受一個型別作為參數。總之，我們知道 `fmap` 接受一個函數，這個函數從一個型別映射到另一個型別，還接受一個 functor 裝有原始的型別，然後會回傳一個 functor 裝有映射後的型別。

如果聽不太懂也沒關係。當我們看幾個範例之後會比較好懂。不過這邊 `fmap` 的型別宣告讓我們想起類似的東西，就是 `map :: (a -> b) -> [a] -> [b]`。

他接受一個函數，這函數把一個型別的東西映射成另一個。還有一串裝有某個型別的 List 變成裝有另一個型別的 List。到這邊聽起來實在太像 functor 了。實際上，`map` 就是針對 List 的 `fmap`。來看看 List 是如何被定義成 `Functor` 的 instance 的。

```haskell
instance Functor [] where
    fmap = map
```

注意到我們不是寫成 `instance Functor [a] where`，因為從 `fmap :: (a -> b) -> f a -> f b` 可以知道 `f` 是一個型別構造子，他接受一個型別。而 `[a]` 則已經是一個具體型別（一個擁有某個型別的 List），其中 `[]` 是一個型別構造子，能接受某個型別而構造出像 `[Int]`、`[String]` 甚至是 `[[String]]` 的具體型別。

對於 List，`fmap` 只不過是 `map`，對 List 操作的時候他們都是一樣的。

```haskell
map :: (a -> b) -> [a] -> [b]
ghci> fmap (*2) [1..3]
[2,4,6]
ghci> map (*2) [1..3]
[2,4,6]
```

至於當我們對空的 List 操作 `map` 或 `fmap` 呢？我們會得到一個空的 List。他把一個型別為 `[a]` 的空的 List 轉成型別為 `[b]` 的空的 List。

可以當作盒子的型別可能就是一個 functor。你可以把 List 想做是一個擁有無限小隔間的盒子。他們可能全部都是空的，已也可能有一部份是滿的其他是空的。所以作為一個盒子會具有什麼性質呢？例如說 `Maybe a`。他表現得像盒子在於他可能什麼東西都沒有，就是 `Nothing`，或是可以裝有一個東西，像是 `"HAHA"`，在這邊就是 `Just "HAHA"`。可以看到 `Maybe` 作為一個 functor 的定義：

```haskell
instance Functor Maybe where
    fmap f (Just x) = Just (f x)
    fmap f Nothing = Nothing
```

注意到我們是寫 `instance Functor Maybe where` 而不是 `instance Functor (Maybe m) where`，就像我們在寫 `YesNo` 時的 `Maybe` 一樣。`Functor` 要的是一個接受一個型別參數的型別構造子而不是一個具體型別。如果你把 `f` 代換成 `Maybe`。`fmap` 就會像 `(a -> b) -> Maybe a -> Maybe b`。但如果你把 `f` 代換成 `(Maybe m)`，那他就會像 `(a -> b) -> Maybe m a -> Maybe m b`，這看起來並不合理，因為 `Maybe` 只接受一個型別參數。

總之，`fmap` 的實作是很簡單的。如果一個空值是 `Nothing`，那他就會回傳 `Nothing`。如果我們 map over 一個空的盒子，我們就會得到一個空的盒子。就像我們 map over 一個空的 List，那我們就會得到一個空的 List。如果他不是一個空值，而是包在 `Just` 中的某個值，那我們就會套用在包在 `Just` 中的值。

```haskell
ghci> fmap (++ " HEY GUYS IM INSIDE THE JUST") (Just "Something serious.")
Just "Something serious. HEY GUYS IM INSIDE THE JUST"
ghci> fmap (++ " HEY GUYS IM INSIDE THE JUST") Nothing
Nothing
ghci> fmap (*2) (Just 200)
Just 400
ghci> fmap (*2) Nothing
Nothing
```

另外 `Tree a` 的型別也可以被 map over 且被定義成 `Functor` 的一個 instance。他可以被想成是一個盒子，而 `Tree` 的型別構造子也剛好接受單一一個型別參數。如果你把 `fmap` 看作是一個特別為 `Tree` 寫的函數，他的型別宣告會長得像這樣 `(a -> b) -> Tree a -> Tree b`。不過我們在這邊會用到遞迴。map over 一棵空的樹會得到一棵空的樹。map over 一棵非空的樹會得到一棵被函數映射過的樹，他的 root 會先被映射，然後左右子樹都分別遞迴地被函數映射。

```haskell
instance Functor Tree where
    fmap f EmptyTree = EmptyTree
    fmap f (Node x leftsub rightsub) =
        Node (f x) (fmap f leftsub) (fmap f rightsub)
```

```haskell
ghci> fmap (*2) EmptyTree
EmptyTree
ghci> fmap (*4) (foldr treeInsert EmptyTree [5,7,3,2,1,7])
Node 28 (Node 4 EmptyTree (Node 8 EmptyTree (Node 12 EmptyTree (Node 20 EmptyTree EmptyTree)))) EmptyTree
```

那 `Either a b` 又如何？他可以是一個 functor 嗎？`Functor` 限制型別構造子只能接受一個型別參數，但 `Either` 卻接受兩個。聰明的你會想到我可以 partial apply `Either`，先餵給他一個參數，並把另一個參數當作 free parameter。來看看 `Either a` 在標準函式庫中是如何被定義的：

```haskell
instance Functor (Either a) where
    fmap f (Right x) = Right (f x)
    fmap f (Left x) = Left x
```

我們在這邊做了些什麼？你可以看到我們把 `Either a` 定義成一個 instance，而不是 `Either`。那是因為 `Either a` 是一個接受單一型別參數的型別構造子，而 `Either` 則接受兩個。如果 `fmap` 是針對 `Either a`，那他的型別宣告就會像是 `(b -> c) -> Either a b -> Either a c`，他又等價於 `(b -> c) -> (Either a) b -> (Either a) c`。在實作中，我們碰到一個 `Right` 的時候會做 `map`，但在碰到 `Left` 的時候卻不這樣做，為什麼呢？如果我們回頭看看 `Either a b` 是怎麼定義的：

```haskell
data Either a b = Left a | Right b
```

如果我們希望對他們兩個都做 `map` 的動作，那 `a` 跟 `b` 必須要是相同的型別。也就是說，如果我們的函數是接受一個字串然後回傳另一個字串，而且 `b` 是字串，`a` 是數字，這樣的情形是不可行的。而且從觀察 `fmap` 的型別也可以知道，當他運作在 `Either` 上的時候，第一個型別參數必須固定，而第二個則可以改變，而其中第一個參數正好就是 `Left` 用的。

我們持續用盒子的比喻也仍然貼切，我們可以把 `Left` 想做是空的盒子在他旁邊寫上錯誤訊息，說明為什麼他是空的。

在 `Data.Map` 中的 Map 也可以被定義成 functor，像是 `Map k v` 的情況下，`fmap` 可以用 `v -> v'` 這樣一個函數來 map over `Map k v`，並回傳 `Map k v'`。

注意到 `'` 在這邊並沒有特別的意思，他只是用來表示他跟另一個東西有點像，只有一點點差別而已。

你可以自己試試看把 `Map k` 變成 `Functor` 的一個 instance。

看過了 `Functor` 這個 typeclass，我們知道 typeclass 可以拿來代表高階的概念。我們也練習過不少 partially applying type 跟定義 instance。在下幾章中，我們也會看到 functor 必須要遵守的定律。

還有一件事就是 functor 應該要遵守一些定律，這樣他們的一些性質才能被保證。如果我們用 `fmap (+1)` 來 map over `[1,2,3,4]`，我們會期望結果會是 `[2,3,4,5]` 而不是反過來變成 `[5,4,3,2]`。如果我們使用 `fmap (\a -> a)`來 map over 一個 list，我們會期待拿回相同的結果。例如說，如果我們給 `Tree` 定義了錯誤的 functor instance，對 tree 使用 `fmap` 可能會導致二元搜尋樹的性質喪失，也就是在 root 左邊的節點不再比 root 小，在 root 右邊的節點不再比 root 大。我們在下面幾章會多談 functor laws。

## Kind

型別構造子接受其他型別作為他的參數，來構造出一個具體型別。這樣的行為會讓我們想到函數，也是接受一個值當作參數，並回傳另一個值。我們也看過型別構造子可以 partially apply （`Either String` 是一個型別構造子，他接受一個型別來構造出一個具體型別，就像 `Either String Int`）。這些都是函數能辦到的事。在這個章節中，對於型別如何被套用到型別構造子上，我們會來看一下正式的定義。就像我們之前是用函數的型別來定義出函數是如何套用值的。如果你看不懂的話，你可以跳過這一章，這不會影響你後續的閱讀。然而如果你搞懂的話，你會對於型別系統有更進一步的了解。

像是 `3`,`"YEAH"` 或是 `takeWhile` 的值他們都有自己的型別（函數也是值的一種，我們可以把他們傳來傳去）型別是一個標籤，值會把他帶著，這樣我們就可以推測出他的性質。但型別也有他們自己的標籤，叫做 kind。kind 是型別的型別。雖然聽起來有點玄妙，不過他的確是個有趣的概念。

那kind可以拿來做什麼呢？我們可以在 ghci 中用 `:k` 來得知一個型別的 kind。

```haskell
ghci> :k Int
Int :: *
```

一個星星代表的是什麼意思？一個 `*` 代表這個型別是具體型別。一個具體型別是沒有任何型別參數，而值只能屬於具體型別。而 `*` 的讀法叫做 star 或是 type。

我們再看看 `Maybe` 的 kind：

```haskell
ghci> :k Maybe
Maybe :: * -> *
```

`Maybe` 的型別構造子接受一個具體型別（像是 `Int`）然後回傳一個具體型別，像是 `Maybe Int`。這就是 kind 告訴我們的資訊。就像 `Int -> Int` 代表這個函數接受 `Int` 並回傳一個 `Int`。`* -> *` 代表這個型別構造子接受一個具體型別並回傳一個具體型別。我們再來對 `Maybe` 套用型別參數後再看看他的 kind 是什麼：

```haskell
ghci> :k Maybe Int
Maybe Int :: *
```

正如我們預期的。我們對 `Maybe` 套用了型別參數後會得到一個具體型別（這就是 `* -> *` 的意思）這跟 `:t isUpper` 還有 `:t isUpper 'A'` 的差別有點類似。`isUpper` 的型別是 `Char -> Bool` 而 `isUpper 'A'` 的型別是 `Bool`。而這兩種型別，都是 `*` 的 kind。

我們對一個型別使用 `:k` 來得到他的 kind。就像我們對值使用 `:t` 來得到的他的型別一樣。就像我們先前說的，型別是值的標籤，而 kind 是型別的標籤。

我們再來看看其他的 kind

```haskell
ghci> :k Either
Either :: * -> * -> *
```

這告訴我們 `Either` 接受兩個具體型別作為參數，並構造出一個具體型別。他看起來也像是一個接受兩個參數並回傳值的函數型別。型別構造子是可以做 curry 的，所以我們也能 partially apply。

```haskell
ghci> :k Either String
Either String :: * -> *
ghci> :k Either String Int
Either String Int :: *
```

當我們希望定義 `Either` 成為 `Functor` 的 instance 的時候，我們必須先 partial apply，因為 `Functor` 預期有一個型別參數，但 `Either` 卻有兩個。也就是說，`Functor` 希望型別的 kind 是 `* -> *`，而我們必須先 partial apply `Either` 來得到 kind `* -> *`，而不是最開始的 `* -> * -> *`。我們再來看看 `Functor` 的定義

```haskell
class Functor f where
    fmap :: (a -> b) -> f a -> f b
```

我們看到 `f` 型別變數是接受一個具體型別且構造出一個具體型別的型別。 知道他構造出具體型別是因為是作為函數參數的型別。 從那裡我們可以推測出一個型別要是屬於 `Functor` 必須是 `* -> *` kind。

現在我們來練習一下。來看看下面這個新定義的 typeclass。

```haskell
class Tofu t where
    tofu :: j a -> t a j
```

這看起來很怪。我們幹嘛要為這個奇怪的 typeclass 定義 instance？我們可以來看看他的 kind 是什麼？由於 `j a` 被當作 `tofu` 這個函數的參數的型別，所以 `j a` 一定是 `*` kind。我們假設 `a` 是 `*` kind，那 `j` 就會是 `* -> *` 的 kind。我們看到 `t` 由於是函數的回傳值，一定是接受兩個型別參數的型別。而知道 `a` 是 `*`，`j` 是 `* -> *`，我們可以推測出 `t`是`* -> (* -> *) -> *`。也就是說他接受一個具體型別 `a`，一個接受單一參數的型別構造子 `j`，然後產生出一個具體型別。

我們再來定義出一個型別具有 `* -> (* -> *) -> *` 的 kind，下面是一種定義的方法：

```haskell
data Frank a b  = Frank {frankField :: b a} deriving (Show)
```

我們怎麼知道這個型別具有 `* -> (* -> *) -> *` 的 kind 呢？ADT 中的欄位是要來塞值的，所以他們必須是 `*` kind。我們假設 `a` 是 `*`，那 `b` 就是接受一個型別參數的 kind `* -> *`。現在我們知道 `a` 跟 `b` 的 kind 了，而他們又是 `Frank` 的型別參數，所以我們知道 `Frank` 會有 `* -> (* -> *) -> *` 的 kind。第一個 `*` 代表 `a`，而 `(* -> *)` 代表 `b`。我們構造些 `Frank` 的值並檢查他們的型別吧：

```haskell
ghci> :t Frank {frankField = Just "HAHA"}
Frank {frankField = Just "HAHA"} :: Frank [Char] Maybe
ghci> :t Frank {frankField = Node 'a' EmptyTree EmptyTree}
Frank {frankField = Node 'a' EmptyTree EmptyTree} :: Frank Char Tree
ghci> :t Frank {frankField = "YES"}
Frank {frankField = "YES"} :: Frank Char []
```

由於 `frankField` 具有 `a b` 的型別。他的值必定有一個類似的型別。他們可能是 `Just "HAHA"`，也就有 `Maybe [Char]` 的型別，或是他們可能是 `['Y','E','S']`，他的型別是 `[Char]`。（如果我們是用自己定義的 List 型別的話，那就會是 `List Char`）。我們看到 `Frank` 值的型別對應到 `Frank` 的 kind。`[Char]` 具有 `*` 的 kind，而 `Maybe` 則是 `* -> *`。由於結果必須是個值，也就是他必須要是具體型別，因使他必須 fully applied，因此每個 `Frank blah blaah` 的值都會是 `*` 的 kind。

要把 `Frank` 定義成 `Tofu` 的 instance 也是很簡單。我們看到 `tofu` 接受 `j a`（例如 `Maybe Int`）並回傳 `t a j`。所以我們將 `Frank` 代入 `t`，就得到 `Frank Int Maybe`。

```haskell
instance Tofu Frank where
    tofu x = Frank x
```

```haskell
ghci> tofu (Just 'a') :: Frank Char Maybe
Frank {frankField = Just 'a'}
ghci> tofu ["HELLO"] :: Frank [Char] []
Frank {frankField = ["HELLO"]}
```

這並不是很有用，但讓我們做了不少型別的練習。再來看看下面的型別：

```haskell
data Barry t k p = Barry { yabba :: p, dabba :: t k }
```

我們想要把他定義成 `Functor` 的 instance。`Functor` 希望是 `* -> *` 的型別，但 `Barry` 並不是那種 kind。那 `Barry` 的 kind 是什麼呢？我們可以看到他接受三個型別參數，所以會是 `something -> something -> something -> *`。`p` 是一個具體型別因此是 `*`。至於 `k`，我們假設他是 `*`，所以 `t` 會是 `* -> *`。現在我們把這些代入 `something`，所以 kind 就變成 `(* -> *) -> * -> * -> *`。我們用 ghci 來檢查一下。

```haskell
ghci> :k Barry
Barry :: (* -> *) -> * -> * -> *
```

我們猜對了！現在要把這個型別定義成 `Functor`，我們必須先 partially apply 頭兩個型別參數，這樣我們就會是 `* -> *` 的 kind。這代表 instance 定義會是 `instance Functor (Barry a b) where`。如果我們看 `fmap` 針對 `Barry` 的型別，也就是把 `f` 代換成 `Barry c d`，那就會是 `fmap :: (a -> b) -> Barry c d a -> Barry c d b`。第三個 `Barry` 的型別參數是對於任何型別，所以我們並不牽扯進他。

```haskell
instance Functor (Barry a b) where
    fmap f (Barry {yabba = x, dabba = y}) = Barry {yabba = f x, dabba = y}
```

我們把 `f` map 到第一個欄位。

在這一個章節中，我們看到型別參數是怎麼運作的，以及正如我們用型別來定義出函數的參數，我們也用 kind 是來定義他。我們看到函數跟型別構造子有許多彼此相像的地方。然而他們是兩個完全不同的東西。當我們在寫一般實用的 Haskell 程式時，你幾乎不會碰到需要動到 kind 的東西，也不需要動腦去推敲 kind。通常你只需要在定義 instance 時 partially apply 你自己的 `* -> *` 或是 `*` 型別，但知道背後運作的原理也是很好的。知道型別本身也有自己的型別也是很有趣的。如果你實在不懂這邊講的東西，也可以繼續閱讀下去。但如果你能理解，那你就會理解 Haskell 型別系統的一大部份。

