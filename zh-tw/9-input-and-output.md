# 輸入與輸出

![](img/dognap.png)

我們已經說明了 Haskell 是一個純粹函數式語言。雖說在命令式語言中我們習慣給電腦執行一連串指令，在函數式語言中我們是用定義東西的方式進行。在 Haskell 中，一個函數不能改變狀態，像是改變一個變數的內容。（當一個函數會改變狀態，我們說這函數是有副作用的。）在 Haskell 中函數唯一可以做的事是根據我們給定的參數來算出結果。如果我們用同樣的參數呼叫兩次同一個函數，它會回傳相同的結果。儘管這從命令式語言的角度來看是蠻大的限制，我們已經看過它可以達成多麼酷的效果。在一個命令式語言中，程式語言沒辦法給你任何保證在一個簡單如打印出幾個數字的函數不會同時燒掉你的房子，綁架你的狗並刮傷你車子的烤漆。例如，當我們要建立一棵二元樹的時候，我們並不插入一個節點來改變原有的樹。由於我們無法改變狀態，我們的函數實際上回傳了一棵新的二元樹。

函數無法改變狀態的好處是它讓我們促進了我們理解程式的容易度，但同時也造成了一個問題。假如說一個函數無法改變現實世界的狀態，那它要如何打印出它所計算的結果？畢竟要告訴我們結果的話，它必須要改變輸出裝置的狀態（譬如說螢幕），然後從螢幕傳達到我們的腦，並改變我們心智的狀態。

不要太早下結論，Haskell 實際上設計了一個非常聰明的系統來處理有副作用的函數，它漂亮地將我們的程式區分成純粹跟非純粹兩部分。非純粹的部分負責跟鍵盤還有螢幕溝通。有了這區分的機制，在跟外界溝通的同時，我們還是能夠有效運用純粹所帶來的好處，像是惰性求值、容錯性跟模組性。

## Hello, world!

![](img/helloworld.png)

到目前為止我們都是將函數載入 GHCi 中來測試，像是標準函式庫中的一些函式。但現在我們要做些不一樣的，寫一個真實跟世界互動的 Haskell 程式。當然不例外，我們會來寫個 "hello world"。

現在，我們把下一行打到你熟悉的編輯器中

```haskell
main = putStrLn "hello, world"
```

我們定義了一個 `main`，並在裡面以 `"hello, world"` 為參數呼叫了 `putStrLn`。看起來沒什麼大不了，但不久你就會發現它的奧妙。把這程式存成 `helloworld.hs`。

現在我們將做一件之前沒做過的事：編譯你的程式。打開你的終端並切換到包含 `helloworld.hs` 的目錄，並輸入下列指令。

```haskell
$ ghc --make helloworld
[1 of 1] Compiling Main                 ( helloworld.hs, hellowowlrd.o )
Linking helloworld ...
```

順利的話你就會得到如上的訊息，接著你便可以執行你的程式 `./helloworld`

```haskell
$ ./helloworld
hello, world
```

這就是我們第一個編譯成功並打印出字串到螢幕的程式。很簡單吧。

讓我們來看一下我們究竟做了些什麼，首先來看一下 `putStrLn` 函數的型態：

```haskell
ghci> :t putStrLn
putStrLn :: String -> IO ()
ghci> :t putStrLn "hello, world"
putStrLn "hello, world" :: IO ()
```

我們可以這麼解讀 `putStrLn` 的型態：`putStrLn` 接受一個字串並回傳一個 I/O action，這 I/O action 包含了 `()` 的型態。（即空的 tuple，或者是 unit 型態）。一個 I/O action 是一個會造成副作用的動作，常是指讀取輸入或輸出到螢幕，同時也代表會回傳某些值。在螢幕打印出幾個字串並沒有什麼有意義的回傳值可言，所以這邊用一個 `()` 來代表。

那究竟 I/O action 會在什麼時候被觸發呢？這就是 `main` 的功用所在。一個 I/O action 會在我們把它綁定到 `main` 這個名字並且執行程式的時候觸發。

把整個程式限制在只能有一個 I/O action 看似是個極大的限制。這就是為什麼我們需要 do 表示法來將所有 I/O action 綁成一個。來看看下面這個例子。

```haskell
main = do
    putStrLn "Hello, what's your name?"
    name <- getLine
    putStrLn ("Hey " ++ name ++ ", you rock!")
```

新的語法，有趣吧！它看起來就像一個命令式的程式。如果你編譯並執行它，它便會照你預期的方式執行。我們寫了一個 do 並且接著一連串指令，就像寫個命令式程式一般，每一步都是一個 I/O action。將所有 I/O action 用 do 綁在一起變成了一個大的 I/O action。這個大的 I/O action 的型態是 `IO ()`，這完全是由最後一個 I/O action 所決定的。

這就是為什麼 `main` 的型態永遠都是 `main :: IO something`，其中 `something` 是某個具體的型態。按照慣例，我們通常不會把 `main` 的型態在程式中寫出來。

另一個有趣的事情是第三行 `name <- getLine`。它看起來像是從輸入讀取一行並存到一個變數 `name` 之中。真的是這樣嗎？我們來看看 `getLine` 的型態吧

```haskell
ghci> :t getLine
getLine :: IO String
```

![](img/luggage.png)

我們可以看到 `getLine` 是一個回傳 `String` 的 I/O action。因為它會等使用者輸入某些字串，這很合理。那 `name <- getLine` 又是如何？你能這樣解讀它：執行一個 I/O action `getLine` 並將它的結果綁定到 `name` 這個名字。`getLine` 的型態是 `IO String`，所以 `name` 的型態會是 `String`。你能把 I/O action 想成是一個長了腳的盒子，它會跑到真實世界中替你做某些事，像是在牆壁上塗鴉，然後帶回來某些資料。一旦它帶了某些資料給你，打開盒子的唯一辦法就是用 `<-`。而且如果我們要從 I/O action 拿出某些資料，就一定同時要在另一個 I/O action 中。這就是 Haskell 如何漂亮地分開純粹跟不純粹的程式的方法。`getLine` 在這樣的意義下是不純粹的，因為執行兩次的時候它沒辦法保證會回傳一樣的值。這也是為什麼它需要在一個 `IO` 的型態建構子中，那樣我們才能在 I/O action 中取出資料。而且任何一段程式一旦依賴著 I/O 資料的話，那段程式也會被視為 I/O code。

但這不表示我們不能在純粹的程式碼中使用 I/O action 回傳的資料。只要我們綁定它到一個名字，我們便可以暫時地使用它。像在 `name <- getLine` 中 `name` 不過是一個普通字串，代表在盒子中的內容。我們能將這個普通的字串傳給一個極度複雜的函數，並回傳你一生會有多少財富。像是這樣：

```haskell
main = do
    putStrLn "Hello, what's your name?"
    name <- getLine
    putStrLn $ "Read this carefully, because this is your future: " ++ tellFortune name
```

`tellFortune` 並不知道任何 I/O 有關的事，它的型態只不過是 `String -> String`。

再來看看這段程式碼吧，他是合法的嗎?

```haskell
nameTag = "Hello, my name is " ++ getLine
```

如果你回答不是，恭喜你。如果你說是，你答錯了。這麼做不對的理由是 `++` 要求兩個參數都必須是串列。他左邊的參數是 `String`，也就是 `[Char]`。然而 `getLine` 的型態是 `IO String`。你不能串接一個字串跟 I/O action。我們必須先把 `String` 的值從 I/O action 中取出，而唯一可行的方法就是在 I/O action 中使用 `name <- getLine`。如果我們需要處理一些非純粹的資料，那我們就要在非純粹的環境中做。所以我們最好把 I/O 的部分縮減到最小的比例。

每個 I/O action 都有一個值封裝在裡面。這也是為什麼我們之前的程式可以這麼寫：

```haskell
main = do
    foo <- putStrLn "Hello, what's your name?"
    name <- getLine
    putStrLn ("Hey " ++ name ++ ", you rock!")
```

然而，`foo` 只會有一個 `()` 的值，所以綁定到 `foo` 這個名字似乎是多餘的。另外注意到我們並沒有綁定最後一行的 `putStrLn` 給任何名字。那是因為在一個 do block 中，最後一個 action 不能綁定任何名字。我們在之後講解 Monad 的時候會說明為什麼。現在你可以先想成 do block 會自動從最後一個 action 取出值並綁定給他的結果。

除了最後一行之外，其他在 do 中沒有綁定名字的其實也可以寫成綁定的形式。所以 `putStrLn "BLAH"` 可以寫成 `_ <- putStrLn "BLAH"`。但這沒什麼實際的意義，所以我們寧願寫成 `putStrLn something`。

初學者有時候會想錯

```haskell
    name = getLine
```

以為這行會讀取輸入並給他綁定一個名字叫 `name` 但其實只是把 `getLine` 這個 I/O action 指定一個名字叫 `name` 罷了。記住，要從一個 I/O action 中取出值，你必須要在另一個 I/O action 中將他用 `<-` 綁定給一個名字。

I/O actions 只會在綁定給 `main` 的時候或是在另一個用 do 串起來的 I/O action 才會執行。你可以用 do 來串接 I/O actions，再用 do 來串接這些串接起來的 I/O actions。不過只有最外面的 I/O action 被指定給 main 才會觸發執行。

喔對，其實還有另外一個情況。就是在 GHCi 中輸入一個 I/O action 並按下 Enter 鍵，那也會被執行

```haskell
ghci> putStrLn "HEEY"
HEEY
```

就算我們只是在 GHCi 中打幾個數字或是呼叫一個函數，按下 Enter 就會計算它並呼叫 `show`，再用 `putStrLn` 將字串打印出在終端上。

還記得 let binding 嗎？如果不記得，回去溫習一下這個章節。它們的形式是 `let bindings in expression`，其中 `bindings` 是 expression 中的名字、`expression` 則是被運用到這些名字的算式。我們也提到了 list comprehensions 中，`in` 的部份不是必需的。你能夠在 do blocks 中使用 let bindings 如同在 list comprehensions 中使用它們一樣，像這樣：

```haskell
import Data.Char

main = do
    putStrLn "What's your first name?"
    firstName <- getLine
    putStrLn "What's your last name?"
    lastName <- getLine
    let bigFirstName = map toUpper firstName
        bigLastName = map toUpper lastName
    putStrLn $ "hey " ++ bigFirstName ++ " " ++ bigLastName ++ ", how are you?"
```

注意我們是怎麼編排在 do block 中的 I/O actions，也注意到我們是怎麼編排 let 跟其中的名字的，由於對齊在 Haskell 中並不會被無視，這麼編排才是好的習慣。我們的程式用 `map toUpper firstName` 將 `"John"` 轉成大寫的 `"JOHN"`，並將大寫的結果綁定到一個名字上，之後在輸出的時候參考到了這個名字。

你也許會問究竟什麼時候要用 `<-`，什麼時候用 let bindings？記住，`<-` 是用來運算 I/O actions 並將他的結果綁定到名稱。而 `map toUpper firstName` 並不是一個 I/O action。他只是一個純粹的 expression。所以總結來說，當你要綁定 I/O actions 的結果時用 `<-`，而對於純粹的 expression 使用 let bindings。對於錯誤的 `let firstName = getLine`，我們只不過是把 `getLine` 這個 I/O actions 給了一個不同的名字罷了。最後還是要用 `<-` 將結果取出。

現在我們來寫一個會一行一行不斷地讀取輸入，並將讀進來的字反過來輸出到螢幕上的程式。程式會在輸入空白行的時候停止。

```haskell
main = do
    line <- getLine
    if null line
        then return ()
        else do
            putStrLn $ reverseWords line
            main

reverseWords :: String -> String
reverseWords = unwords . map reverse . words
```

在分析這段程式前，你可以執行看看來感受一下程式的運行。

首先，我們來看一下 `reverseWords`。他不過是一個普通的函數，假如接受了個字串 `"hey there man"`，他會先呼叫 `words` 來產生一個字的串列 `["hey", "there", "man"]`。然後用 `reverse` 來 map 整個串列，得到 `["yeh", "ereht", "nam"]`，接著用 `unwords` 來得到最終的結果 `"yeh ereht nam"`。這些用函數合成來簡潔的表達。如果沒有用函數合成，那就會寫成醜醜的樣子 `reverseWords st = unwords (map reverse (words st))`

那 `main` 又是怎麼一回事呢？首先，我們用 `getLine` 從終端讀取了一行，並把這行輸入取名叫 `line`。然後接著一個條件式 expression。記住，在 Haskell 中 if 永遠要伴隨一個 else，這樣每個 expression 才會有值。當 if 的條件是 true （也就是輸入了一個空白行），我們便執行一個 I/O action，如果 if 的條件是 false，那 else 底下的 I/O action 被執行。這也就是說當 if 在一個 I/O do block 中的時候，長的樣子是 `if condition then I/O action else I/O action`。

我們首先來看一下在 else 中發生了什麼事。由於我們在 else 中只能有一個 I/O action，所以我們用 do 來將兩個 I/O actions 綁成一個，你可以寫成這樣：

```haskell
else (do
    putStrLn $ reverseWords line
    main)
```

這樣可以明顯看到整個 do block 可以看作一個 I/O action，只是比較醜。但總之，在 do block 裡面，我們依序呼叫了 `getLine` 以及 `reverseWords`，在那之後，我們遞迴呼叫了 `main`。由於 main 也是一個 I/O action，所以這不會造成任何問題。呼叫 `main` 也就代表我們回到程式的起點。

那假如 `null line` 的結果是 true 呢？也就是說 then 的區塊被執行。我們看一下區塊裡面有 `then return ()`。如果你是從 C、Java 或 Python 過來的，你可能會認為 `return` 不過是作一樣的事情便跳過這一段。但很重要的： `return` 在 Hakell 裡面的意義跟其他語言的 `return` 完全不同！他們有相同的樣貌，造成了許多人搞錯，但確實他們是不一樣的。在命令式語言中，`return` 通常結束 method 或 subroutine 的執行，並且回傳某個值給呼叫者。在 Haskell 中，他的意義則是利用某個 pure value 造出 I/O action。用之前盒子的比喻來說，就是將一個 value 裝進箱子裡面。產生出的 I/O action 並沒有作任何事，只不過將 value 包起來而已。所以在 I/O 的情況下來說，`return "haha"` 的型態是 `IO String`。將 pure value 包成 I/O action 有什麼實質意義呢？為什麼要弄成 `IO` 包起來的值？這是因為我們一定要在 else 中擺上某些 I/O action，所以我們才用 `return ()` 做了一個沒作什麼事情的 I/O action。

在 I/O do block 中放一個 `return` 並不會結束執行。像下面這個程式會執行到底。

```haskell
main = do
    return ()
    return "HAHAHA"
    line <- getLine
    return "BLAH BLAH BLAH"
    return 4
    putStrLn line
```

所有在程式中的 `return` 都是將 value 包成 I/O actions，而且由於我們沒有將他們綁定名稱，所以這些結果都被忽略。我們能用 `<-` 與 `return` 來達到綁定名稱的目的。

```haskell
main = do
    a <- return "hell"
    b <- return "yeah!"
    putStrLn $ a ++ " " ++ b
```

可以看到 `return` 與 `<-` 作用相反。`return` 把 value 裝進盒子中，而 `<-` 將 value 從盒子拿出來，並綁定一個名稱。不過這麼做是有些多餘，因為你可以用 let bindings 來綁定

```haskell
main = do
    let a = "hell"
        b = "yeah"
    putStrLn $ a ++ " " ++ b
```

在 I/O do block 中需要 `return` 的原因大致上有兩個：一個是我們需要一個什麼事都不做的 I/O action，或是我們不希望這個 do block 形成的 I/O action 的結果值是這個 block 中的最後一個 I/O action，我們希望有一個不同的結果值，所以我們用 `return` 來作一個 I/O action 包了我們想要的結果放在 do block 的最後。

在我們接下去講檔案之前，讓我們來看看有哪些實用的函數可以處理 I/O。

`putStr` 跟 `putStrLn` 幾乎一模一樣，都是接受一個字串當作參數，並回傳一個 I/O action 打印出字串到終端上，只差在 `putStrLn` 會換行而 `putStr` 不會罷了。

```haskell
main = do putStr "Hey, "
          putStr "I'm "
          putStrLn "Andy!"
```

```haskell
$ runhaskell putstr_test.hs
Hey, I'm Andy!
```

他的 type signature 是 `putStr :: String -> IO ()`，所以是一個包在 I/O action 中的 unit。也就是空值，沒有辦法綁定他。

`putChar` 接受一個字元，並回傳一個 I/O action 將他打印到終端上。

```haskell
main = do putChar 't'
          putChar 'e'
          putChar 'h'
```

```haskell
$ runhaskell putchar_test.hs
teh
```

`putStr` 實際上就是 `putChar` 遞迴定義出來的。`putStr` 的邊界條件是空字串，所以假設我們打印一個空字串，那他只是回傳一個什麼都不做的 I/O action，像 `return ()`。如果打印的不是空字串，那就先用 `putChar` 打印出字串的第一個字元，然後再用 `putStr` 打印出字串剩下部份。

```haskell
putStr :: String -> IO ()
putStr [] = return ()
putStr (x:xs) = do
    putChar x
    putStr xs
```

看看我們如何在 I/O 中使用遞迴，就像我們在 pure code 中所做的一樣。先定義一個邊界條件，然後再思考剩下如何作。

`print` 接受任何是 `Show` typeclass 的 instance 的型態的值，這代表我們知道如何用字串表示他，呼叫 `show` 來將值變成字串然後將其輸出到終端上。基本上，他就是 `putStrLn . show`。首先呼叫 `show` 然後把結果餵給 `putStrLn`，回傳一個 I/O action 打印出我們的值。

```haskell
main = do print True
          print 2
          print "haha"
          print 3.2
          print [3,4,3]
```

```haskell
$ runhaskell print_test.hs
True
2
"haha"
3.2
[3,4,3]
```

就像你看到的，這是個很方便的函數。還記得我們提到 I/O actions 只有在 `main` 中才會被執行以及在 GHCI 中運算的事情嗎？當我們用鍵盤打了些值，像 `3` 或 `[1,2,3]` 並按下 Enter，GHCI 實際上就是用了 `print` 來將這些值輸出到終端。

```haskell
ghci> 3
3
ghci> print 3
3
ghci> map (++"!") ["hey","ho","woo"]
["hey!","ho!","woo!"]
ghci> print (map (++"!") ["hey", "ho", "woo"])
["hey!","ho!","woo!"]
```

當我們需要打印出字串，我們會用 `putStrLn`，因為我們不想要周圍有引號，但對於輸出值來說，`print` 才是最常用的。

`getChar` 是一個從輸入讀進一個字元的 I/O action，因此他的 type signature 是 `getChar :: IO Char`，代表一個 I/O action 的結果是 `Char`。注意由於緩衝區的關係，只有當 Enter 被按下的時候才會觸發讀取字元的行為。

```haskell
main = do
    c <- getChar
    if c /= ' '
        then do
            putChar c
            main
        else return ()
```

這程式看起來像是讀取一個字元並檢查他是否為一個空白。如果是的話便停止，如果不是的話便打印到終端上並重複之前的行為。在某種程度上來說也不能說錯，只是結果不如你預期而已。來看看結果吧。

```haskell
$ runhaskell getchar_test.hs
hello sir
hello
```

上面的第二行是輸入。我們輸入了 `hello sir` 並按下了 Enter。由於緩衝區的關係，程式是在我們按了 Enter 後才執行而不是在某個輸入字元的時候。一旦我們按下了 Enter，那他就把我們直到目前輸入的一次做完。

`when` 這函數可以在 `Control.Monad` 中找到他 \(你必須 `import Contorl.Monad` 才能使用他\)。他在一個 do block 中看起來就像一個控制流程的 statement，但實際上他的確是一個普通的函數。他接受一個 boolean 值跟一個 I/O action。如果 boolean 值是 `True`，便回傳我們傳給他的 I/O action。如果 boolean 值是 `False`，便回傳 `return ()`，即什麼都不做的 I/O action。我們接下來用 `when` 來改寫我們之前的程式。

```haskell
import Control.Monad

main = do
    c <- getChar
    when (c /= ' ') $ do
        putChar c
        main
```

就像你看到的，他可以將 `if something then do some I/O action else return ()` 這樣的模式封裝起來。

`sequence` 接受一串 I/O action，並回傳一個會依序執行他們的 I/O action。運算的結果是包在一個 I/O action 的一連串 I/O action 的運算結果。他的 type signature 是 `sequence :: [IO a] -> IO [a]`

```haskell
main = do
    a <- getLine
    b <- getLine
    c <- getLine
    print [a,b,c]
```

其實可以寫成

```haskell
main = do
    rs <- sequence [getLine, getLine, getLine]
    print rs
```

所以 `sequence [getLine, getLine, getLine]` 作成了一個執行 `getLine` 三次的 I/O action。如果我們對他綁定一個名字，結果便是這串結果的串列。也就是說，三個使用者輸入的東西組成的串列。

一個常見的使用方式是我們將 `print` 或 `putStrLn` 之類的函數 map 到串列上。`map print [1,2,3,4]` 這個動作並不會產生一個 I/O action，而是一串 I/O action，就像是 `[print 1, print 2, print 3, print 4]`。如果我們將一串 I/O action 變成一個 I/O action，我們必須用 `sequence`

```haskell
ghci> sequence (map print [1,2,3,4,5])
1
2
3
4
5
[(),(),(),(),()]
```

那 `[(),(),(),(),()]` 是怎麼回事？當我們在 GHCI 中運算 I/O action，他會被執行並把結果打印出來，唯一例外是結果是 `()` 的時候不會被打印出。這也是為什麼 `putStrLn "hehe"` 在 GHCI 中只會打印出 `hehe`（因為 `putStrLn "hehe"` 的結果是 `()`）。但當我們使用 `getLine` 時，由於 `getLine` 的型態是 `IO String`，所以結果會被打印出來。

由於對一個串列 map 一個回傳 I/O action 的函數，然後再 sequence 他這個動作太常用了。所以有一些函數在函式庫中 `mapM` 跟 `mapM_`。`mapM` 接受一個函數跟一個串列，將對串列用函數 map 然後 sequence 結果。`mapM_` 也作同樣的事，只是他把運算的結果丟掉而已。在我們不關心 I/O action 結果的情況下，`mapM_` 是最常被使用的。

```haskell
ghci> mapM print [1,2,3]
1
2
3
[(),(),()]
ghci> mapM_ print [1,2,3]
1
2
3
```

`forever` 接受一個 I/O action 並回傳一個永遠作同一件事的 I/O action。你可以在 `Control.Monad` 中找到他。下面的程式會不斷地要使用者輸入些東西，並把輸入的東西轉成大寫輸出到螢幕上。

```haskell
import Control.Monad
import Data.Char

main = forever $ do
    putStr "Give me some input: "
    l <- getLine
    putStrLn $ map toUpper l
```

在 `Control.Monad` 中的 `forM` 跟 `mapM` 的作用一樣，只是參數的順序相反而已。第一個參數是串列，而第二個則是函數。這有什麼用？在一些有趣的情況下還是有用的：

```haskell
import Control.Monad

main = do
    colors <- forM [1,2,3,4] (\a -> do
        putStrLn $ "Which color do you associate with the number " ++ show a ++ "?"
        color <- getLine
        return color)
    putStrLn "The colors that you associate with 1, 2, 3 and 4 are: "
    mapM putStrLn colors
```

`(\a -> do ...)` 是接受一個數字並回傳一個 I/O action 的函數。我們必須用括號括住他，不然 lambda 會貪心 match 的策略會把最後兩個 I/O action 也算進去。注意我們在 do block 裡面 `return color`。我們那麼作是讓 do block 的結果是我們選的顏色。實際上我們並不需那麼作，因為 `getLine` 已經達到我們的目的。先 `color <- getLine` 再 `return color` 只不過是把值取出再包起來，其實是跟 `getLine` 效果相當。`forM` 產生一個 I/O action，我們把結果綁定到 `colors` 這名稱。`colors` 是一個普通包含字串的串列。最後，我們用 `mapM putStrLn colors` 打印出所有顏色。

你可以把 `forM` 的意思想成將串列中的每個元素作成一個 I/O action。至於每個 I/O action 實際作什麼就要看原本的元素是什麼。然後，執行這些 I/O action 並將結果綁定到某個名稱上。或是直接將結果忽略掉。

```haskell
$ runhaskell from_test.hs
Which color do you associate with the number 1?
white
Which color do you associate with the number 2?
blue
Which color do you associate with the number 3?
red
Which color do you associate with the number 4?
orange
The colors that you associate with 1, 2, 3 and 4 are:
white
blue
red
orange
```

其實我們也不是一定要用到 `forM`，只是用了 `forM` 程式會比較容易理解。正常來講是我們需要在 map 跟 sequence 的時候定義 I/O action 的時候使用 `forM`，同樣地，我們也可以將最後一行寫成 `forM colors putStrLn`。

在這一節，我們學會了輸入與輸出的基礎。我們也了解了什麼是 I/O action，他們是如何幫助我們達成輸入與輸出的目的。這邊重複一遍，I/O action 跟其他 Haskell 中的 value 沒有兩樣。我們能夠把他當參數傳給函式，或是函式回傳 I/O action。他們特別之處在於當他們是寫在 `main` 裡面或 GHCI 裡面的時候，他們會被執行，也就是實際輸出到你螢幕或輸出音效的時候。每個 I/O action 也能包著一個從真實世界拿回來的值。

不要把像是 `putStrLn` 的函式想成接受字串並輸出到螢幕。要想成一個函式接受字串並回傳一個 I/O action。當 I/O action 被執行的時候，會漂亮地打印出你想要的東西。

## 檔案與字符流

![](img/streams.png)

`getChar` 是一個讀取單一字元的 I/O action。`getLine` 是一個讀取一行的 I/O action。這是兩個非常直覺的函式，多數程式語言也有類似這兩個函式的 statement 或 function。但現在我們來看看 _getContents_。`getContents` 是一個從標準輸入讀取直到 end-of-file 字元的 I/O action。他的型態是 `getContents :: IO String`。最酷的是 `getContents` 是惰性 I/O \(Lazy I/O\)。當我們寫了 `foo <- getContents`，他並不會馬上讀取所有輸入，將他們存在 memory 裡面。他只有當你真的需要輸入資料的時候才會讀取。

當我們需要重導一個程式的輸出到另一個程式的輸入時，`getContents` 非常有用。假設我們有下面一個文字檔：

```haskell
I'm a lil' teapot
What's with that airplane food, huh?
It's so small, tasteless
```

還記得我們介紹 `forever` 時寫的小程式嗎？會把所有輸入的東西轉成大寫的那一個。為了防止你忘記了，這邊再重複一遍。

```haskell
import Control.Monad
import Data.Char

main = forever $ do
    putStr "Give me some input: "
    l <- getLine
    putStrLn $ map toUpper l
```

將我們的程式存成 `capslocker.hs` 然後編譯他。然後用 Unix 的 Pipe 將文字檔餵給我們的程式。我們使用的是 GNU 的 cat，會將指定的檔案輸出到螢幕。

```haskell
$ ghc --make capslocker
[1 of 1] Compiling Main             ( capslocker.hs, capslocker.o )
Linking capslocker ...
$ cat haiku.txt
I'm a lil' teapot
What's with that airplane food, huh?
It's so small, tasteless
$ cat haiku.txt | ./capslocker
I'M A LIL' TEAPOT
WHAT'S WITH THAT AIRPLANE FOOD, HUH?
IT'S SO SMALL, TASTELESS
capslocker <stdin>: hGetLine: end of file
```

就如你看到的，我們是用 `|` 這符號來將某個程式的輸出 piping 到另一個程式的輸入。我們做的事相當於 run 我們的 capslocker，然後將 haiku 的內容用鍵盤打到終端上，最後再按 Ctrl-D 來代表 end-of-file。這就像執行 cat haiku.txt 後大喊，嘿，不要把內容打印到終端上，把內容塞到 capslocker！

我們用 `forever` 在做的事基本上就是將輸入經過轉換後變成輸出。用 `getContents` 的話可以讓我們的程式更加精鍊。

```haskell
import Data.Char

main = do
    contents <- getContents
    putStr (map toUpper contents)
```

我們將 `getContents` 取回的字串綁定到 `contents`。然後用 `toUpper` map 到整個字串後打印到終端上。記住字串基本上就是一串惰性的串列 \(list\)，同時 `getContents` 也是惰性 I/O，他不會一口氣讀入內容然後將內容存在記憶體中。實際上，他會一行一行讀入並輸出大寫的版本，這是因為輸出才是真的需要輸入的資料的時候。

```haskell
$ cat haiku.txt | ./capslocker
I'M A LIL' TEAPOT
WHAT'S WITH THAT AIRPLAN FOOD, HUH?
IT'S SO SMALL, TASTELESS
```

很好，程式運作正常。假如我們執行 capslocker 然後自己打幾行字呢？

```haskell
$ ./capslocker
hey ho
HEY HO
lets go
LETS GO
```

按下 Ctrl-D 來離開環境。就像你看到的，程式是一行一行將我們的輸入打印出來。當 `getContent` 的結果被綁定到 `contents` 的時候，他不是被表示成在記憶體中的一個字串，反而比較像是他有一天會是字串的一個承諾。當我們將 `toUpper` map 到 `contents` 的時候，便也是一個函數被承諾將會被 map 到內容上。最後 `putStr` 則要求先前的承諾說，給我一行大寫的字串吧。實際上還沒有任何一行被取出，所以便跟 `contents` 說，不如從終端那邊取出些字串吧。這才是 `getContents` 真正從終端讀入一行並把這一行交給程式的時候。程式便將這一行用 `toUpper` 處理並交給 `putStr`，`putStr` 則打印出他。之後 `putStr` 再說：我需要下一行。整個步驟便再重複一次，直到讀到 end-of-file 為止。

接著我們來寫個程式，讀取輸入，並只打印出少於十個字元的行。

```haskell
main = do
    contents <- getContents
    putStr (shortLinesOnly contents)

shortLinesOnly :: String -> String
shortLinesOnly input =
    let allLines = lines input
        shortLines = filter (\line -> length line < 10) allLines
        result = unlines shortLines
    in result
```

我們把 I/O 部份的程式碼弄得很短。由於程式的行為是接某些輸入，作些處理然後輸出。我們可以把他想成讀取輸入，呼叫一個函數，然後把函數的結果輸出。

`shortLinesOnly` 的行為是這樣：拿到一個字串，像是 `"short\nlooooooooooooooong\nshort again"`。這字串有三行，前後兩行比較短，中間一行很常。他用 `lines` 把字串分成 `["short", "looooooooooooooong", "short again"]`，並把結果綁定成 `allLines`。然後過濾這些字串，只有少於十個字元的留下，`["short", "short again"]`，最後用 `unlines` 把這些字串用換行接起來，形成 `"short\nshort again"`

```haskell
i'm short
so am i
i am a loooooooooong line!!!
yeah i'm long so what hahahaha!!!!!!
short line
loooooooooooooooooooooooooooong
short
```

```haskell
$ ghc --make shortlinesonly
[1 of 1] Compiling Main             ( shortlinesonly.hs, shortlinesonly.o )
Linking shortlinesonly ...
$ cat shortlines.txt | ./shortlinesonly
i'm short
so am i
short
```

我們把 shortlines.txt 的內容經由 pipe 送給 shortlinesonly，結果就如你看到，我們只有得到比較短的行。

從輸入那一些字串，經由一些轉換然後輸出這樣的模式實在太常用了。常用到甚至建立了一個函數叫 **interact**。`interact` 接受一個 `String -> String` 的函數，並回傳一個 I/O action。那個 I/O action 會讀取一些輸入，呼叫提供的函數，然後把函數的結果打印出來。所以我們的程式可以改寫成這樣。

```haskell
main = interact shortLinesOnly

shortLinesOnly :: String -> String
shortLinesOnly input =
    let allLines = lines input
        shortLines = filter (\line -> length line < 10) allLines
        result = unlines shortLines
    in result
```

我們甚至可以再讓程式碼更短一些，像這樣

```haskell
main = interact $ unlines . filter ((<10) . length) . lines
```

看吧，我們讓程式縮到只剩一行了，很酷吧！

能應用 `interact` 的情況有幾種，像是從輸入 pipe 讀進一些內容，然後丟出一些結果的程式；或是從使用者獲取一行一行的輸入，然後丟回根據那一行運算的結果，再拿取另一行。這兩者的差別主要是取決於使用者使用他們的方式。

我們再來寫另一個程式，它不斷地讀取一行行並告訴我們那一行字串是不是一個回文字串 \(palindrome\)。我們當然可以用 `getLine` 讀取一行然後再呼叫 `main` 作同樣的事。不過同樣的事情可以用 `interact` 更簡潔地達成。當使用 `interact` 的時候，想像你是將輸入經有某些轉換成輸出。在這個情況當中，我們要將每一行輸入轉換成 `"palindrome"` 或 `"not a palindrome"`。所以我們必須寫一個函數將 `"elephant\nABCBA\nwhatever"` 轉換成 `not a palindrome\npalindrome\nnot a palindrome"`。來動手吧！

```haskell
respondPalindromes contents = unlines (map (\xs ->
    if isPalindrome xs then "palindrome" else "not a palindrome") (lines contents))
        where isPalindrome xs = xs == reverse xs
```

再來將程式改寫成 point-free 的形式

```haskell
respondPalindromes = unlines . map (\xs ->
    if isPalindrome xs then "palindrome" else "not a palindrome") . lines
        where isPalindrome xs = xs == reverse xs
```

很直覺吧！首先將 `"elephant\nABCBA\nwhatever"` 變成 `["elephant", "ABCBA", "whatever"]` 然後將一個 lambda 函數 map 它，`["not a palindrome", "palindrome", "not a palindrome"]` 然後用 `unlines` 變成一行字串。接著

```haskell
main = interact respondPalindromes
```

來測試一下吧。

```haskell
$ runhaskell palindrome.hs
hehe
not a palindrome
ABCBA
palindrome
cookie
not a palindrome
```

即使我們的程式是把一大把字串轉換成另一個，其實他表現得好像我們是一行一行做的。這是因為 Haskell 是惰性的，程式想要打印出第一行結果時，他必須要先有第一行輸入。所以一旦我們給了第一行輸入，他便打印出第一行結果。我們用 end-of-line 字元來結束程式。

我們也可以用 pipe 的方式將輸入餵給程式。假設我們有這樣一個檔案。

```haskell
dogaroo
radar
rotor
madam
```

將他存為 `words.txt`，將他餵給程式後得到的結果

```haskell
$ cat words.txt | runhaskell palindromes.hs
not a palindrome
palindrome
palindrome
palindrome
```

再一次地提醒，我們得到的結果跟我們自己一個一個字打進輸入的內容是一樣的。我們看不到 `palindrome.hs` 輸入的內容是因為內容來自於檔案。

你應該大致了解 Lazy I/O 是如何運作，並能善用他的優點。他可以從輸入轉換成輸出的角度方向思考。由於 Lazy I/O，沒有輸入在被用到之前是真的被讀入。

到目前為止，我們的示範都是從終端讀取某些東西或是打印出某些東西到終端。但如果我們想要讀寫檔案呢？其實從某個角度來說我們已經作過這件事了。我們可以把讀寫終端想成讀寫檔案。只是把檔案命名成 `stdout` 跟 `stdin` 而已。他們分別代表標準輸出跟標準輸入。我們即將看到的讀寫檔案跟讀寫終端並沒什麼不同。

首先來寫一個程式，他會開啟一個叫 girlfriend.txt 的檔案，檔案裡面有 Avril Lavigne 的暢銷名曲 Girlfriend，並將內容打印到終端上。接下來是 girlfriend.txt 的內容。

```haskell
Hey! Hey! You! You!
I don't like your girlfriend!
No way! No way!
I think you need a new one!
```

這則是我們的主程式。

```haskell
import System.IO

main = do
    handle <- openFile "girlfriend.txt" ReadMode
    contents <- hGetContents handle
    putStr contents
    hClose handle
```

執行他後得到的結果。

```haskell
$ runhaskell girlfriend.hs
Hey! Hey! You! You!
I don't like your girlfriend!
No way! No way!
I think you need a new one!
```

我們來一行行看一下程式。我們的程式用 do 把好幾個 I/O action 綁在一起。在 do block 的第一行，我們注意到有一個新的函數叫 **openFile**。他的 type signature 是 `openFile :: FilePath -> IOMode -> IO Handle`。他說了 `openFile` 接受一個檔案路徑跟一個 `IOMode`，並回傳一個 I/O action，他會打開一個檔案並把檔案關聯到一個 handle。

`FilePath` 不過是 `String` 的 type synonym。

```haskell
type FilePath = String
```

`IOMode` 則是一個定義如下的型態

```haskell
data IOMode = ReadMode | WriteMode | AppendMode | ReadWriteMode
```

![](img/file.png)

就像我們之前定義的型態，分別代表一個星期的七天。這個型態代表了我們想對打開的檔案做什麼。很簡單吧。留意到我們的型態是 `IOMode` 而不是 `IO Mode`。`IO Mode` 代表的是一個 I/O action 包含了一個型態為 `Mode` 的值，但 `IOMode` 不過是一個陽春的 enumeration。

最後，他回傳一個 I/O action 會將指定的檔案用指定的模式打開。如果我們將 I/O action 綁定到某個東西，我們會得到一個 `Handle`。型態為 `Handle` 的值代表我們的檔案在哪裡。有了 handle 我們才知道要從哪個檔案讀取內容。想讀取檔案但不將檔案綁定到 handle 上這樣做是很蠢的。所以，我們將一個 handle 綁定到 `handle`。

接著一行，我們看到一個叫 **hGetContents** 的函數。他接了一個 `Handle`，所以他知道要從哪個檔案讀取內容並回傳一個 `IO String`。一個包含了檔案內容的 I/O action。這函數跟 `getContents` 差不多。唯一的差別是 `getContents` 會自動從標準輸入讀取內容（也就是終端），而 `hGetContents` 接了一個 file handle，這 file handle 告訴他讀取哪個檔案。除此之外，他們都是一樣的。就像 `getContents`，`hGetContents` 不會把檔案一次都拉到記憶體中，而是有必要才會讀取。這非常酷，因為我們把 `contents` 當作是整個檔案般用，但他實際上不在記憶體中。就算這是個很大的檔案，`hGetContents` 也不會塞爆你的記憶體，而是只有必要的時候才會讀取。

要留意檔案的 handle 還有檔案的內容兩個概念的差異，在我們的程式中他們分別被綁定到 `handle` 跟 `contents` 兩個名字。handle 是我們拿來區分檔案的依據。如果你把整個檔案系統想成一本厚厚的書，每個檔案分別是其中的一個章節，handle 就像是書籤一般標記了你現在正在閱讀（或寫入）哪一個章節，而內容則是章節本身。

我們使用 `putStr contents` 打印出內容到標準輸出，然後我們用了 **hClose**。他接受一個 handle 然後回傳一個關掉檔案的 I/O action。在用了 `openFile` 之後，你必須自己把檔案關掉。

要達到我們目的的另一種方式是使用 **withFile**，他的 type signature 是 `withFile :: FilePath -> IOMode -> (Handle -> IO a) -> IO a`。他接受一個檔案路徑，一個 `IOMode` 以及一個函數，這函數則接受一個 handle 跟一個 I/O action。`withFile` 最後回傳一個會打開檔案，對檔案作某件事然後關掉檔案的 I/O action。處理的結果是包在最後的 I/O action 中，這結果跟我們給的函數的回傳是相同的。這聽起來有些複雜，但其實很簡單，特別是我們有 lambda，來看看我們用 `withFile` 改寫前面程式的一個範例：

```haskell
import System.IO

main = do
    withFile "girlfriend.txt" ReadMode (\handle -> do
            contents <- hGetContents handle
            putStr contents)
```

正如你看到的，程式跟之前的看起來很像。`(\handle -> ... )` 是一個接受 handle 並回傳 I/O action 的函數，他通常都是用 lambda 來表示。我們需要一個回傳 I/O action 的函數的理由而不是一個本身作處理並關掉檔案的 I/O action，是因為這樣一來那個 I/O action 不會知道他是對哪個檔案在做處理。用 `withFile` 的話，`withFile` 會打開檔案並把 handle 傳給我們給他的函數，之後他則拿到一個 I/O action，然後作成一個我們描述的 I/O action，最後關上檔案。例如我們可以這樣自己作一個 `withFile`：

```haskell
withFile' :: FilePath -> IOMode -> (Handle -> IO a) -> IO a
withFile' path mode f = do
    handle <- openFile path mode
    result <- f handle
    hClose handle
    return result
```

![](img/edd.png)

我們知道要回傳的是一個 I/O action，所以我們先放一個 do。首先我們打開檔案，得到一個 handle。然後我們 apply `handle` 到我們的函數，並得到一個做事的 I/O action。我們綁定那個 I/O action 到 `result` 這個名字，關上 handle 並 `return result`。`return` 的作用把從 `f` 得到的結果包在 I/O action 中，這樣一來 I/O action 中就包含了 `f handle` 得到的結果。如果 `f handle` 回傳一個從標準輸入讀去數行並寫到檔案然後回傳讀入的行數的 I/O action，在 `withFile'` 的情形中，最後的 I/O action 就會包含讀入的行數。

就像 `hGetContents` 對應 `getContents` 一樣，只不過是針對某個檔案。我們也有 **hGetLine**、**hPutStr**、**hPutStrLn**、**hGetChar** 等等。他們分別是少了 h 的那些函數的對應。只不過他們要多拿一個 handle 當參數，並且是針對特定檔案而不是標準輸出或標準輸入。像是 `putStrLn` 是一個接受一個字串並回傳一個打印出加了換行字元的字串的 I/O action 的函數。`hPutStrLn` 接受一個 handle 跟一個字串，回傳一個打印出加了換行字元的字串到檔案的 I/O action。以此類推，`hGetLine` 接受一個 handle 然後回傳一個從檔案讀取一行的 I/O action。

讀取檔案並對他們的字串內容作些處理實在太常見了，常見到我們有三個函數來更進一步簡化我們的工作。

**readFile** 的 type signature 是 `readFile :: FilePath -> IO String`。記住，`FilePath` 不過是 `String` 的一個別名。`readFile` 接受一個檔案路徑，回傳一個惰性讀取我們檔案的 I/O action。然後將檔案的內容綁定到某個字串。他比起先 `openFile`，綁定 handle，然後 `hGetContents` 要好用多了。這邊是一個用 `readFile` 改寫之前例子的範例：

```haskell
import System.IO

main = do
    contents <- readFile "girlfriend.txt"
    putStr contents
```

由於我們拿不到 handle，所以我們也無法關掉他。這件事 Haskell 的 `readFile` 在背後幫我們做了。

**writeFile** 的型態是 `writefile :: FilePath -> String -> IO ()`。他接受一個檔案路徑，以及一個要寫到檔案中的字串，並回傳一個寫入動作的 I/O action。如果這個檔案已經存在了，他會先把檔案內容都砍了再寫入。下面示範了如何把 girlfriend.txt 的內容轉成大寫然後寫入到 girlfriendcaps.txt 中

```haskell
import System.IO
import Data.Char

main = do
    contents <- readFile "girlfriend.txt"
    writeFile "girlfriendcaps.txt" (map toUpper contents)
```

```haskell
$ runhaskell girlfriendtocaps.hs
$ cat girlfriendcaps.txt
HEY! HEY! YOU! YOU!
I DON'T LIKE YOUR GIRLFRIEND!
NO WAY! NO WAY!
I THINK YOU NEED A NEW ONE!
```

**appendFile** 的型態很像 `writeFile`，只是 `appendFile` 並不會在檔案存在時把檔案內容砍掉而是接在後面。

假設我們有一個檔案叫 todo.txt\`\`，裡面每一行是一件要做的事情。現在我們寫一個程式，從標準輸入接受一行將他加到我們的 to-do list 中。

```haskell
import System.IO

main = do
    todoItem <- getLine
    appendFile "todo.txt" (todoItem ++ "\n")
```

```haskell
$ runhaskell appendtodo.hs
Iron the dishes
$ runhaskell appendtodo.hs
Dust the dog
$ runhaskell appendtodo.hs
Take salad out of the oven
$ cat todo.txt
Iron the dishes
Dust the dog
Take salad out of the oven
```

由於 `getLine` 回傳的值不會有換行字元，我們需要在每一行最後加上 `"\n"`。

還有一件事，我們提到 `contents <- hGetContents handle` 是惰性 I/O，不會將檔案一次都讀到記憶體中。 所以像這樣寫的話：

```haskell
main = do
    withFile "something.txt" ReadMode (\handle -> do
        contents <- hGetContents handle
        putStr contents)
```

實際上像是用一個 pipe 把檔案弄到標準輸出。正如你可以把 list 想成 stream 一樣，你也可以把檔案想成 stream。他會每次讀一行然後打印到終端上。你也許會問這個 pipe 究竟一次可以塞多少東西，讀去硬碟的頻率究竟是多少？對於文字檔而言，預設的 buffer 通常是 line-buffering。這代表一次被讀進來的大小是一行。這也是為什麼在這個 case 我們是一行一行處理。對於 binary file 而言，預設的 buffer 是 block-buffering。這代表我們是一個 chunk 一個 chunk 去讀得。而一個 chunk 的大小是根據作業系統不同而不同。

你能用 `hSetBuffering` 來控制 buffer 的行為。他接受一個 handle 跟一個 `BufferMode`，回傳一個會設定 buffer 行為的 I/O action。`BufferMode` 是一個 enumeration 型態，他可能的值有：`NoBuffering`, `LineBuffering` 或 `BlockBuffering (Maybe Int)`。其中 `Maybe Int` 是表示一個 chunck 有幾個 byte。如果他的值是 `Nothing`，則作業系統會幫你決定 chunk 的大小。`NoBuffering` 代表我們一次讀一個 character。一般來說 `NoBuffering` 的表現很差，因為他存取硬碟的頻率很高。

接下來是我們把之前的範例改寫成用 2048 bytes 的 chunk 讀取，而不是一行一行讀。

```haskell
main = do
    withFile "something.txt" ReadMode (\handle -> do
        hSetBuffering handle $ BlockBuffering (Just 2048)
        contents <- hGetContents handle
        putStr contents)
```

用更大的 chunk 來讀取對於減少存取硬碟的次數是有幫助的，特別是我們的檔案其實是透過網路來存取。

我們也可以使用 **hFlush**，他接受一個 handle 並回傳一個會 flush buffer 到檔案的 I/O action。當我們使用 line-buffering 的時候，buffer 在每一行都會被 flush 到檔案。當我們使用 block-buffering 的時候，是在我們讀每一個 chunk 作 flush 的動作。flush 也會發生在關閉 handle 的時候。這代表當我們碰到換行字元的時候，讀或寫的動作都會停止並回報手邊的資料。但我們能使用 `hFlush` 來強迫回報所有已經在 buffer 中的資料。經過 flushing 之後，資料也就能被其他程式看見。

把 block-buffering 的讀取想成這樣：你的馬桶會在水箱有一加侖的水的時候自動沖水。所以你不斷灌水進去直到一加侖，馬桶就會自動沖水，在水裡面的資料也就會被看到。但你也可以手動地按下沖水鈕來沖水。他會讓現有的水被沖走。沖水這個動作就是 `hFlush` 這個名字的含意。

我們已經寫了一個將 item 加進 to-do list 裡面的程式，現在我們想加進移除 item 的功能。我先把程式碼貼上然後講解他。我們會使用一些新面孔像是 `System.Directory` 以及 `System.IO` 裡面的函數。

來看一下我們包含移除功能的程式:

```haskell
import System.IO
import System.Directory
import Data.List

main = do
    handle <- openFile "todo.txt" ReadMode
    (tempName, tempHandle) <- openTempFile "." "temp"
    contents <- hGetContents handle
    let todoTasks = lines contents
    numberedTasks = zipWith (\n line -> show n ++ " - " ++ line) [0..] todoTasks
    putStrLn "These are your TO-DO items:"
    putStr $ unlines numberedTasks
    putStrLn "Which one do you want to delete?"
    numberString <- getLine
    let number = read numberString
    newTodoItems = delete (todoTasks !! number) todoTasks
    hPutStr tempHandle $ unlines newTodoItems
    hClose handle
    hClose tempHandle
    removeFile "todo.txt"
    renameFile tempName "todo.txt"
```

一開始，我們用 read mode 打開 todo.txt，並把他綁定到 `handle`。

接著，我們使用了一個之前沒用過在 `System.IO` 中的函數 **openTempFile**。他的名字淺顯易懂。他接受一個暫存的資料夾跟一個樣板檔案名，然後打開一個暫存檔。我們使用 `"."` 當作我們的暫存資料夾，因為 `.` 在幾乎任何作業系統中都代表了現在所在的資料夾。我們使用 `"temp"` 當作我們暫存檔的樣板名，他代表暫存檔的名字會是 temp 接上某串隨機字串。他回傳一個創建暫存檔的 I/O action，然後那個 I/O action 的結果是一個 pair：暫存檔的名字跟一個 handle。我們當然可以隨便開啟一個 todo2.txt 這種名字的檔案。但使用 `openTempFile` 會是比較好的作法，這樣你不會不小心覆寫任何檔案。

我們不用 `getCurrentDirectory` 的來拿到現在所在資料夾而用 `"."` 的原因是 `.` 在 unix-like 系統跟 Windows 中都表示現在的資料夾。

然後，我們綁定 todo.txt 的內容成 `contents`。把字串斷成一串字串，每個字串代表一行。`todoTasks` 就變成 `["Iron the dishes", "Dust the dog", "Take salad out of the oven"]`。我們用一個會把 3 跟 `"hey"` 變成 `"3 - hey"` 的函數，然後從 0 開始把這個串列 zip 起來。所以 `numberedTasks` 就是 `["0 - Iron the dishes", "1 - Dust the dog" ...`。我們用 `unlines` 把這個串列變成一行，然後打印到終端上。注意我們也有另一種作法，就是用 `mapM putStrLn numberedTasks`。

我們問使用者他們想要刪除哪一個並且等著他們輸入一個數字。假設他們想要刪除 1 號，那代表 `Dust the dog`，所以他們輸入 `1`。於是 `numberString` 就代表 `"1"`。由於我們想要一個數字，而不是一個字串，所以我們用對 `1` 使用 `read`，並且綁定到 `number`。

還記得在 `Data.List` 中的 `delete` 跟 `!!` 嗎？`!!` 回傳某個 index 的元素，而 `delete` 刪除在串列中第一個發現的元素，然後回傳一個新的沒有那個元素的串列。`(todoTasks !! number)` （number 代表 `1`） 回傳 `"Dust the dog"`。我們把 `todoTasks` 去掉第一個 `"Dust the dog"` 後的串列綁定到 `newTodoItems`，然後用 `unlines` 變成一行然後寫到我們所打開的暫存檔。舊有的檔案並沒有變動，而暫存檔包含砍掉那一行後的所有內容。

在我們關掉原始檔跟暫存檔之後我們用 **removeFile** 來移除原本的檔案。他接受一個檔案路徑並且刪除檔案。刪除舊得 todo.txt 之後，我們用 **renameFile** 來將暫存檔重新命名成 todo.txt。特別留意 `removeFile` 跟 `renameFile`（兩個都在 `System.Directory` 中）接受的是檔案路徑，而不是 handle。

這就是我們要的，實際上我們可以用更少行寫出同樣的程式，但我們很小心地避免覆寫任何檔案，並詢問作業系統我們可以把暫存檔擺在哪？讓我們來執行看看。

```haskell
$ runhaskell deletetodo.hs
These are your TO-DO items:
0 - Iron the dishes
1 - Dust the dog
2 - Take salad out of the oven
Which one do you want to delete?
1

$ cat todo.txt
Iron the dishes
Take salad out of the oven

$ runhaskell deletetodo.hs
These are your TO-DO items:
0 - Iron the dishes
1 - Take salad out of the oven
Which one do you want to delete?
0

$ cat todo.txt
Take salad out of the oven
```

## 命令列引數

![](img/arguments.png)

如果你想要寫一個在終端裡運行的程式，處理命令列引數是不可或缺的。幸運的是，利用 Haskell 的 Standard Libary 能讓我們有效地處理命令列引數。

在之前的章節中，我們寫了一個能將 to-do item 加進或移除 to-do list 的一個程式。但我們的寫法有兩個問題。第一個是我們把放 to-do list 的檔案名稱給寫死了。我們擅自決定使用者不會有很多個 to-do lists，就把檔案命名為 todo.txt。

一種解決的方法是每次都詢問使用者他們想將他們的 to-do list 放進哪個檔案。我們在使用者要刪除的時候也採用這種方式。這是一種可以運作的方式，但不太能被接受，因為他需要使用者運行程式，等待程式詢問才能回答。這被稱為互動式的程式，但討厭的地方在當你想要自動化執行程式的時候，好比說寫成 script，這會讓你的 script 寫起來比較困難。

這也是為什麼有時候讓使用者在執行的時候就告訴程式他們要什麼會比較好，而不是讓程式去問使用者要什麼。比較好的方式是讓使用者透過命令列引數告訴程式他們想要什麼。

在 `System.Environment` 模組當中有兩個很酷的 I/O actions，一個是 **getArgs**，他的 type 是 `getArgs :: IO [String]`，他是一個拿取命令列引數的 I/O action，並把結果放在包含的一個串列中。**getProgName** 的型態是 `getProgName :: IO String`，他則是一個 I/O action 包含了程式的名稱。

我們來看一個展現他們功能的程式。

```haskell
import System.Environment
import Data.List

main = do
    args <- getArgs
    progName <- getProgName
    putStrLn "The arguments are:"
    mapM putStrLn args
    putStrLn "The program name is:"
    putStrLn progName
```

我們將 `getArgs` 跟 `progName` 分別綁定到 `args` 跟 `progName`。我們打印出 `The arguments are:` 以及在 `args` 中的每個引數。最後，我們打印出程式的名字。我們把程式編譯成 `arg-test`。

```haskell
$ ./arg-test first second w00t "multi word arg"
The arguments are:
first
second
w00t
multi word arg
The program name is:
arg-test
```

知道了這些函數現在你能寫幾個很酷的命令列程式。在之前的章節，我們寫了一個程式來加入待作事項，也寫了另一個程式刪除事項。現在我們要把兩個程式合起來，他會根據命令列引數來決定該做的事情。我們也會讓程式可以處理不同的檔案，而不是只有 todo.txt

我們叫這程式 todo，他會作三件事：

```text
# 檢視待作事項
# 加入待作事項
# 刪除待作事項
```

我們暫不考慮不合法的輸入這件事。

我們的程式要像這樣運作：假如我們要加入 `Find the magic sword of power`，則我們會打 `todo add todo.txt "Find the magic sword of power"`。要檢視事項我們則會打 `todo view todo.txt`，如果要移除事項二則會打 `todo remove todo.txt 2`

我們先作一個分發的 association list。他會把命令列引數當作 key，而對應的處理函數當作 value。這些函數的型態都是 `[String] -> IO ()`。他們會接受命令列引數的串列並回傳對應的檢視，加入以及刪除的 I/O action。

```haskell
import System.Environment
import System.Directory
import System.IO
import Data.List

dispatch :: [(String, [String] -> IO ())]
dispatch =  [ ("add", add)
            , ("view", view)
            , ("remove", remove)
            ]
```

我們定義了 `main`，`add`，`view` 跟 `remove`，就從 `main` 開始講吧：

```haskell
main = do
    (command:args) <- getArgs
    let (Just action) = lookup command dispatch
    action args
```

首先，我們取出引數並把他們綁定到 `(command:args)`。如果你還記得 pattern matching，這麼做會把第一個引數綁定到 `command`，把其他的綁定到 `args`。如果我們像這樣執行程式 `todo add todo.txt "Spank the monkey"`，`command` 會變成 `"add"`，而 `args` 會變成 `["todo.txt", "Spank the monkey"]`。

在下一行，我們在一個分派的串列中尋到我們的指令是哪個。由於 `"add"` 指向 `add`，我們的結果便是 `Just add`。我們再度使用了 pattern matching 來把我們的函數從 `Maybe` 中取出。但如果我們想要的指令不在分派的串列中呢？那樣 lookup 就會回傳 `Nothing`，但我們這邊並不特別處理失敗的情況，所以 pattern matching 會失敗然後我們的程式就會當掉。

最後，我們用剩下的引數呼叫 `action` 這個函數。他會還傳一個加入 item，顯示所有 items 或者刪除 item 的 I/O action。由於這個 I/O action 是在 `main` 的 do block 中，他最後會被執行。如果我們的 `action` 函數是 `add`，他就會被餵 `args` 然後回傳一個加入 `Spank the monkey` 到 todo.txt 中的 I/O action。

我們剩下要做的就是實作 `add`，`view` 跟 `remove`，我們從 `add` 開始：

```haskell
add :: [String] -> IO ()
add [fileName, todoItem] = appendFile fileName (todoItem ++ "\n")
```

如果我們這樣執行程式 `todo add todo.txt "Spank the monkey"`，則 `"add"` 會被綁定到 `command`，而 `["todo.txt", "Spank the monkey"]` 會被帶到從 dispatch list 中拿到的函數。

由於我們不處理不合法的輸入，我們只針對這兩項作 pattern matching，然後回傳一個附加一行到檔案末尾的 I/O action。

接著，我們來實作檢視串列。如果我們想要檢視所有 items，我們會 `todo view todo.txt`。所以 `command` 會是 `"view"`，而 `args` 會是 `["todo.txt"]`。

```haskell
view :: [String] -> IO ()
view [fileName] = do
    contents <- readFile fileName
    let todoTasks = lines contents
    numberedTasks = zipWith (\n line -> show n ++ " - " ++ line) [0..] todoTasks
    putStr $ unlines numberedTasks
```

這跟我們之前刪除檔案的程式差不多，只是我們是在顯示內容而已，

最後，我們要來實作 `remove`。他基本上跟之前寫的只有刪除功能的程式很像，所以如果你不知道刪除是怎麼做的，可以去看之前的解釋。主要的差別是我們不寫死 todo.txt，而是從參數取得。我們也不會提示使用者要刪除哪一號的 item，而是從參數取得。

```haskell
remove :: [String] -> IO ()
remove [fileName, numberString] = do
    handle <- openFile fileName ReadMode
    (tempName, tempHandle) <- openTempFile "." "temp"
    contents <- hGetContents handle
    let number = read numberString
        todoTasks = lines contents
        newTodoItems = delete (todoTasks !! number) todoTasks
    hPutStr tempHandle $ unlines newTodoItems
    hClose handle
    hClose tempHandle
    removeFile fileName
    renameFile tempName fileName
```

我們打開 `fileName` 的檔案以及一個暫存。刪除使用者要我們刪的那一行後，把檔案內容寫到暫存檔。砍掉原本的檔案然後把暫存檔重新命名成 `fileName`。

來看看完整的程式。

```haskell
import System.Environment
import System.Directory
import System.IO
import Data.List

dispatch :: [(String, [String] -> IO ())]
dispatch =  [ ("add", add)
            , ("view", view)
            , ("remove", remove)
            ]

main = do
    (command:args) <- getArgs
    let (Just action) = lookup command dispatch
    action args

add :: [String] -> IO ()
add [fileName, todoItem] = appendFile fileName (todoItem ++ "\n")

view :: [String] -> IO ()
view [fileName] = do
    contents <- readFile fileName
    let todoTasks = lines contents
        numberedTasks = zipWith (\n line -> show n ++ " - " ++ line) [0..] todoTasks
    putStr $ unlines numberedTasks

remove :: [String] -> IO ()
remove [fileName, numberString] = do
    handle <- openFile fileName ReadMode
    (tempName, tempHandle) <- openTempFile "." "temp"
    contents <- hGetContents handle
    let number = read numberString
        todoTasks = lines contents
        newTodoItems = delete (todoTasks !! number) todoTasks
    hPutStr tempHandle $ unlines newTodoItems
    hClose handle
    hClose tempHandle
    removeFile fileName
    renameFile tempName fileName
```

![](img/salad.png)

總結我們的程式：我們做了一個 dispatch association，將指令對應到一些會接受命令列引數並回傳 I/O action 的函數。我們知道使用者下了什麼命令，並根據那個命令從 dispatch list 取出對影的函數。我們用剩下的命令列引數呼叫哪些函數而得到一些作相對應事情的 I/O action。然後便執行那些 I/O action。

在其他程式語言，我們可能會用一個大的 switch case 來實作，但使用高階函數讓我們可以要 dispatch list 給我們要的函數，並要那些函數給我們適當的 I/O action。

讓我們看看執行結果。

```haskell
$ ./todo view todo.txt
0 - Iron the dishes
1 - Dust the dog
2 - Take salad out of the oven

$ ./todo add todo.txt "Pick up children from drycleaners"

$ ./todo view todo.txt
0 - Iron the dishes
1 - Dust the dog
2 - Take salad out of the oven
3 - Pick up children from drycleaners

$ ./todo remove todo.txt 2

$ ./todo view todo.txt
0 - Iron the dishes
1 - Dust the dog
2 - Pick up children from drycleaners
```

要再另外加新的選項也是很容易。只要在 dispatch list 加入新的會作你要的事情函數。你可以試試實作一個 `bump` 函數，接受一個檔案跟一個 task number，他會回傳一個把那個 task 搬到 to-do list 頂端的 I/O action。

對於不合法的輸入你也可以讓程式結束地漂亮一點。\(例如使用者輸入了 `todo UP YOURS HAHAHAHA`\)可以作一個回報錯誤的 I/O action \(例如 \`\`errorExist :: IO \(\)\)檢查有沒有不合法的輸入，如果有便執行這個回報錯誤的 I/O action。我們之後會談另一個可能，就是用 exception。

## 亂數

![](img/random.png)

在許多情況下，你寫程式會需要些隨機的資料。或許你在製作一個遊戲，在遊戲中你需要擲骰子。或是你需要測試程式的測試資料。精準一點地說，我們需要 pseudo-random 的資料，我們知道真正的隨機資料好比是一隻猴子拿著起司跟奶油騎在單輪車上，任何事情都會發生。在這個章節，我們要看看如何讓 Haskell 產生些 pseudo-random 的資料。

在大多數其他的程式語言中，會給你一些函數能讓你拿到些隨機亂數。每呼叫一次他就會拿到一個不同的數字。那在 Haskell 中是如何？要記住 Haskell 是一個純粹函數式語言。代表任何東西都具有 referential transparency。那代表你餵給一個函數相同的參數，不管怎麼呼叫都是回傳相同的結果。這很新奇的原因是因為他讓我們理解程式的方式不同，而且可以讓我們延遲計算，直到我們真正需要他。如果我呼叫一個函數，我可以確定他不會亂來。我真正在乎的是他的結果。然而，這會造成在亂數的情況有點複雜。如果我有一個函數像這樣：

```haskell
randomNumber :: (Num a) => a
randomNumber = 4
```

由於他永遠回傳 `4`，所以對於亂數的情形而言是沒什麼意義。就算 4 這個結果是擲骰子來的也沒有意義。

其他的程式語言是怎麼產生亂數的呢？他們可能隨便拿取一些電腦的資訊，像是現在的時間，你怎麼移動你的滑鼠，以及周圍的聲音。根據這些算出一個數值讓他看起來好像隨機的。那些要素算出來的結果可能在每個時間都不同，所以你會拿到不同的隨機數字。

所以說在 Haskell 中，假如我們能作一個函數，他會接受一個具隨機性的參數，然後根據那些資訊還傳一個數值。

在 `System.Random` 模組中。他包含所有滿足我們需求的函數。讓我們先來看其中一個，就是 **random**。他的型態是 `random :: (RandomGen g, Random a) => g -> (a, g)`。哇，出現了新的 typeclass。**RandomGen** typeclass 是指那些可以當作亂源的型態。而**Random** typeclass 則是可以裝亂數的型態。一個布林值可以是隨機值，不是 `True` 就是 `False`。一個整數可以是隨機的好多不同值。那你會問，函數可以是一個隨機值嗎？我不這麼認為。如果我們試著翻譯 `random` 的型態宣告，大概會是這樣：他接受一個 random generator \(亂源所在\)，然後回傳一個隨機值以及一個新的 random generator。為什麼他要回傳一個新的 random generator 呢？就是下面我們要講的。

要使用 `random` 函數， 我們必須要了解 random generator。 在 `System.Random` 中有一個很酷的型態，叫做 **StdGen**， 他是 `RandomGen` 的一個 instance。 我們可以自己手動作一個 `StdGen` 也可以告訴系統給我們一個現成的。

要自己做一個 random generator，要使用 **mkStdGen** 這個函數。他的型態是 `mkStdGen :: Int -> StdGen`。他接受一個整數，然後根據這個整數會給一個 random generator。讓我們來試一下 `random` 以及 `mkStdGen`，用他們產生一個亂數吧。

```haskell
ghci> random (mkStdGen 100)
```

```haskell
<interactive>:1:0:
    Ambiguous type variable `a' in the constraint:
        `Random a' arising from a use of `random' at <interactive>:1:0-20
    Probable fix: add a type signature that fixes these type variable(s)  `
```

這是什麼？由於 `random` 函數會回傳 `Random` typeclass 中任何一種型態，所以我們必須告訴 Haskell 我們是要哪一種型態。不要忘了我們是回傳 random value 跟 random generator 的一個 pair

```haskell
ghci> random (mkStdGen 100) :: (Int, StdGen)
(-1352021624,651872571 1655838864)
```

我們終於有了一個看起來像亂數的數字。tuple 的第一個部份是我們的亂數，而第二個部份是一個新的 random generator 的文字表示。如果我們用相同的 random generator 再呼叫 `random` 一遍呢？

```haskell
ghci> random (mkStdGen 100) :: (Int, StdGen)
(-1352021624,651872571 1655838864)
```

不易外地我們得到相同的結果。所以我們試試用不同的 random generator 作為我們的參數。

```haskell
ghci> random (mkStdGen 949494) :: (Int, StdGen)
(539963926,466647808 1655838864)
```

很好，我們拿到了不同的數字。我們可以用不同的型態標誌來拿到不同型態的亂數

```haskell
ghci> random (mkStdGen 949488) :: (Float, StdGen)
(0.8938442,1597344447 1655838864)
ghci> random (mkStdGen 949488) :: (Bool, StdGen)
(False,1485632275 40692)
ghci> random (mkStdGen 949488) :: (Integer, StdGen)
(1691547873,1597344447 1655838864)
```

讓我們寫一個模擬丟三次銅板的函數。假如 `random` 不同時回傳一個亂數以及一個新的 random generator，我們就必須讓這函數接受三個 random generators 讓他們每個回傳一個擲銅板的結果。但那樣聽起來怪怪的，加入一個 generator 可以產生一個型態是 `Int` 的亂數，他應該可以產生擲三次銅板的結果（總共才八個組合）。這就是 `random` 為什麼要回傳一個新的 generator 的關鍵了。

我們將一個銅板表示成 `Bool`。`True` 代表反面，`False` 代表正面。

```haskell
threeCoins :: StdGen -> (Bool, Bool, Bool)
threeCoins gen =
    let (firstCoin, newGen) = random gen
    (secondCoin, newGen') = random newGen
    (thirdCoin, newGen') = random newGen'
    in  (firstCoin, secondCoin, thirdCoin)  )
```

我們用我們拿來當參數的 generator 呼叫 `random` 並得到一個擲銅板的結果跟一個新的 generator。然後我們再用新的 generator 呼叫他一遍，來得到第二個擲銅板的結果。對於第三個擲銅板的結果也是如法炮製。如果我們一直都用同樣的 generator，那所有的結果都會是相同的值。也就是不是 `(False, False, False)` 就是 `(True, True, True)`。

```haskell
ghci> threeCoins (mkStdGen 21)
(True,True,True)
ghci> threeCoins (mkStdGen 22)
(True,False,True)
ghci> threeCoins (mkStdGen 943)
(True,False,True)
ghci> threeCoins (mkStdGen 944)
(True,True,True)
```

留意我們不需要寫 `random gen :: (Bool, StdGen)`。那是因為我們已經在函數的型態宣告那邊就表明我們要的是布林。而 Haskell 可以推敲出我們要的是布林值。

假如我們要的是擲四次？甚至五次呢？有一個函數叫 **randoms**，他接受一個 generator 並回傳一個無窮序列。

```haskell
ghci> take 5 $ randoms (mkStdGen 11) :: [Int]
[-1807975507,545074951,-1015194702,-1622477312,-502893664]
ghci> take 5 $ randoms (mkStdGen 11) :: [Bool]
[True,True,True,True,False]
ghci> take 5 $ randoms (mkStdGen 11) :: [Float]
[7.904789e-2,0.62691015,0.26363158,0.12223756,0.38291094]
```

為什麼 `randoms` 不另外多回傳一個新的 generator 呢？我們可以這樣地實作 `randoms`

```haskell
randoms' :: (RandomGen g, Random a) => g -> [a]
randoms' gen = let (value, newGen) = random gen in value:randoms' newGen
```

一個遞迴的定義。我們由現在的 generator 拿到一個亂數跟一個新的 generator，然後製作一個 list，list 的第一個值是那個亂數，而 list 的其餘部份是根據新的 generator 產生出的其餘亂數們。由於我們可能產生出無限的亂數，所以不可能回傳一個新的 generator。

我們可以寫一個函數，他會回傳有限個亂數跟一個新的 generator

```haskell
finiteRandoms :: (RandomGen g, Random a, Num n, Eq n) => n -> g -> ([a], g)
finiteRandoms 0 gen = ([], gen)
finiteRandoms n gen =
    let (value, newGen) = random gen
        (restOfList, finalGen) = finiteRandoms (n-1) newGen
    in  (value:restOfList, finalGen)
```

又是一個遞迴的定義。我們說如果我們要 0 個亂數，我們便回傳一個空的 list 跟原本給我們的 generator。對於其他數量的亂數，我們先拿一個亂數跟一個新的 generator。這一個亂數便是 list 的第一個數字。然後 list 中剩下的便是 n-1 個由新的 generator 產生出的亂數。然後我們回傳整個 list 跟最後一個產生完 n-1 個亂數後 generator。

如果我們要的是在某個範圍內的亂數呢？現在拿到的亂數要不是太大就是太小。如果我們想要的是骰子上的數字呢？**randomR** 能滿足我們的需求。他的型態是 `randomR :: (RandomGen g, Random a) :: (a, a) -> g -> (a, g)`，代表他有點類似 `random`。只不過他的第一個參數是一對數目，定義了最後產生亂數的上界以及下界。

```haskell
ghci> randomR (1,6) (mkStdGen 359353)
(6,1494289578 40692)
ghci> randomR (1,6) (mkStdGen 35935335)
(3,1250031057 40692)
```

另外也有一個 **randomRs** 的函數，他會產生一連串在給定範圍內的亂數：

```haskell
ghci> take 10 $ randomRs ('a','z') (mkStdGen 3) :: [Char]
"ndkxbvmomg"
```

這結果看起來像是一個安全性很好的密碼。

你會問你自己，這一單元跟 I/O 有關系嗎？到現在為止還沒出現任何跟 I/O 有關的東西。到現在為止我們都是手動地做我們的 random generator。但那樣的問題是，程式永遠都會回傳同樣的亂數。這在真實世界中的程式是不能接受的。這也是為什麼 `System.Random` 要提供 **getStdGen** 這個 I/O action，他的型態是 `IO StdGen`。當你的程式執行時，他會跟系統要一個 random generator，並存成一個 global generator。`getStdGen` 會替你拿那個 global random generator 並把他綁定到某個名稱上。

這裡有一個簡單的產生隨機字串的程式。

```haskell
import System.Random

main = do
    gen <- getStdGen
    putStr $ take 20 (randomRs ('a','z') gen)
```

```haskell
$ runhaskell random_string.hs
pybphhzzhuepknbykxhe
$ runhaskell random_string.hs
eiqgcxykivpudlsvvjpg
$ runhaskell random_string.hs
nzdceoconysdgcyqjruo
$ runhaskell random_string.hs
bakzhnnuzrkgvesqplrx
```

要當心當我們連續兩次呼叫 `getStdGent` 的時候，實際上都會回傳同樣的 global generator。像這樣：

```haskell
import System.Random

main = do
    gen <- getStdGen
    putStrLn $ take 20 (randomRs ('a','z') gen)
    gen2 <- getStdGen
    putStr $ take 20 (randomRs ('a','z') gen2)
```

你會打印出兩次同樣的字串。要能得到兩個不同的字串是建立一個無限的 stream，然後拿前 20 個字當作第一個字串，拿下 20 個字當作第二個字串。要這麼做，我們需要在 `Data.List` 中的 `splitAt` 函數。他會把一個 list 根據給定的 index 切成一個 tuple，tuple 的第一部份就是切斷的前半，第二個部份就是切斷的後半。

```haskell
import System.Random
import Data.List

main = do
    gen <- getStdGen
    let randomChars = randomRs ('a','z') gen
        (first20, rest) = splitAt 20 randomChars
        (second20, _) = splitAt 20 rest
    putStrLn first20
    putStr second20
```

另一種方法是用 **newStdGen** 這個 I/O action，他會把現有的 random generator 分成兩個新的 generators。然後會把其中一個指定成 global generator，並回傳另一個。

```haskell
import System.Random

main = do
    gen <- getStdGen
    putStrLn $ take 20 (randomRs ('a','z') gen)
    gen' <- newStdGen
    putStr $ take 20 (randomRs ('a','z') gen')
```

當我們綁定 `newStdGen` 的時候我們不只是會拿到一個新的 generator，global generator 也會被重新指定。所以再呼叫一次 `getStdGen` 並綁定到某個名稱的話，我們就會拿到跟 `gen` 不一樣的 generator。

這邊有一個小程式會讓使用者猜數字：

```haskell
import System.Random
import Control.Monad(when)

main = do
    gen <- getStdGen
    askForNumber gen

askForNumber :: StdGen -> IO ()
askForNumber gen = do
    let (randNumber, newGen) = randomR (1,10) gen :: (Int, StdGen)
    putStr "Which number in the range from 1 to 10 am I thinking of? "
    numberString <- getLine
    when (not $ null numberString) $ do
        let number = read numberString
        if randNumber == number
            then putStrLn "You are correct!"
            else putStrLn $ "Sorry, it was " ++ show randNumber
            askForNumber newGen
```

![](img/jackofdiamonds.png)

我們寫了一個 `askForNumber` 的函數，他接受一個 random generator 並回傳一個問使用者要數字並回答是否正確的 I/O action。在那個函數裡面，我們先根據從參數拿到的 generator 產生一個亂數以及一個新的 generator，分別叫他們為 `randomNumber` 跟 `newGen`。假設那個產生的數字是 `7`。則我們要求使用者猜我們握有的數字是什麼。我們用 `getLine` 來將結果綁定到 `numberString` 上。當使用者輸入 `7`，`numberString` 就會是 `"7"`。接下來，我們用 `when` 來檢查使用者輸入的是否是空字串。如果是，那一個空的 I/O action `return ()` 就會被回傳。基本上就等於是結束程式的意思。如果不是，那 I/O action 就會被執行。我們用 `read` 來把 `numberString` 轉成一個數字，所以 `number` 便會是 `7`。

```text
如果使用者給我們一些 ``read`` 沒辦法讀取的輸入（像是 ``"haha"``），我們的程式便會當掉並打印出錯誤訊息。 如果你不希望你的程式當掉，就用 **reads**，當讀取失敗的時候他會回傳一個空的 list。當成功的時候他就回傳一個 tuple，第一個部份是我們想要的數字，第二個部份是讀取失敗的字串。
```

我們檢查如果輸入的數字跟我們隨機產生的數字一樣，便提示使用者恰當的訊息。然後再遞迴地呼叫 `askForNumber`，只是會拿到一個新的 generator。就像之前的 generator 一樣，他會給我們一個新的 I/O action。

`main` 的組成很簡單，就是由拿取一個 random generator 跟呼叫 `askForNumber` 組成罷了。

來看看我們的程式：

```haskell
$ runhaskell guess_the_number.hs
Which number in the range from 1 to 10 am I thinking of? 4
Sorry, it was 3
Which number in the range from 1 to 10 am I thinking of? 10
You are correct!
Which number in the range from 1 to 10 am I thinking of? 2
Sorry, it was 4
Which number in the range from 1 to 10 am I thinking of? 5
Sorry, it was 10
Which number in the range from 1 to 10 am I thinking of?
```

用另一種方式寫的話像這樣：

```haskell
import System.Random
import Control.Monad(when)

main = do
    gen <- getStdGen
    let (randNumber, _) = randomR (1,10) gen :: (Int, StdGen)
    putStr "Which number in the range from 1 to 10 am I thinking of? "
    numberString <- getLine
    when (not $ null numberString) $ do
        let number = read numberString
        if randNumber == number
            then putStrLn "You are correct!"
            else putStrLn $ "Sorry, it was " ++ show randNumber
        newStdGen
        main
```

他非常類似我們之前的版本，只是不是遞迴地呼叫，而是把所有的工作都在 `main` 裡面做掉。在告訴使用者他們猜得是否正確之後，便更新 global generator 然後再一次呼叫 `main`。兩種策略都是有效但我比較喜歡第一種方式。因為他在 `main` 裡面做的事比較少，並提供我們一個可以重複使用的函數。

## Bytestrings

![](img/chainchomp.png)

List 是一種有用又酷的資料結構。到目前為止，我們幾乎無處不使用他。有好幾個函數是專門處理 List 的，而 Haskell 惰性的性質又讓我們可以用 filter 跟 map 來替換其他語言中的 for loop 跟 while loop。也由於 evaluation 只會發生在需要的時候，像 infinite list 也對於 Haskell 不成問題（甚至是 infinite list of infinite list）。這也是為什麼 list 能被用來表達 stream，像是讀取標準輸入或是讀取檔案。我們可以打開檔案然後讀取內容成字串，即便實際上我們是需要的時候才會真正取讀取。

然而，用字串來處理檔案有一個缺點：就是他很慢。就像你所知道的，`String` 是一個 `[Char]` 的 type synonym。`Char` 沒有一個固定的大小，因為他可能由好幾個 byte 組成，好比說 Unicode。再加上 list 是惰性的。如果你有一個 list 像 `[1,2,3,4]`，他只會在需要的時候被 evaluate。所以整個 list 其實比較像是一個"保證"你會有一個 list。要記住 `[1,2,3,4]` 不過是 `1:2:3:4:[]` 的一個 syntactic sugar。當 list 的第一個元素被 evaluated 的時候，剩餘的部份 `2:3:4:[]` 一樣也只是一個"保證"你會有一個 list，以此類推。以此類推。以此類推。所以你可以想像成 list 是保證在你需要的時候會給你第一個元素，以及保證你會有剩下的部份當你還需要更多的時候。其實不難說服你這樣做並不是一個最有效率的作法。

這樣額外的負擔在大多數時候不會造成困擾，但當我們要讀取一個很大的檔案的時候就是個問題了。這也是為什麼 Haskell 要有 `bytestrings`。Bytestrings 有點像 list，但他每一個元素都是一個 byte \(8 bits\)，而且他們惰性的程度也是不同。

Bytestrings 有兩種：strict 跟 lazy。Strict bytestrings 放在 `Data.ByteString`，他們把惰性的性質完全拿掉。不會有所謂任何的「保證」，一個 strict bytestring 就代表一連串的 bytes。因此你不會有一個無限長的 strict bytestrings。如果你 evaluate 第一個 byte，你就必須 evalute 整個 bytestring。這麼做的優點是他會比較少 overhaed，因為他沒有 "Thunk"（也就是用 Haskell 術語來說的「保證」）。缺點就是他可能會快速消耗你的記憶體，因為你把他們一次都讀進了記憶體。

另一種 bytestring 是放在 `Data.ByteString.Lazy` 中。他們具有惰性，但又不像 list 那麼極端。就像我們之前說的，List 的 thunk 個數是跟 list 中有幾個元素一模一樣。這也是為什麼他們速度沒辦法滿足一些特殊需求。Lazy bytestrings 則用另一種作法，他們被存在 chunks 中（不要跟 Thunk 搞混），每一個 chunk 的大小是 64K。所以如果你 evaluate lazy bytestring 中的 byte，則前 64K 會被 evaluated。在那個 chunck 之後，就是一些「保證」會有剩餘的 chunk。lazy bytestrings 有點像裝了一堆大小為 64K 的 strict bytestrings 的 list。當你用 lazy bytestring 處理一個檔案的時候，他是一個 chunk 一個 chunk 去讀。這很棒是因為他不會讓我們一下使用大量的記憶體，而且 64K 有很高的可能性能夠裝進你 CPU 的 L2 Cache。

如果你大概看過 `Data.ByteString.Lazy` 的文件，你會看到到他有一堆函數的名稱跟 `Data.List` 中的函數名稱相同，只是出現的 type signature 是 `ByteString` 而不是 `[a]`，是 `Word8` 而不是 `a`。同樣名稱的函數基本上表現的行為跟 list 中的差不多。因為名稱是一樣的，所以必須用 qualified import 才不會在裝載進 GHCI 的時候造成衝突。

```haskell
import qualified Data.ByteString.Lazy as B
import qualified Data.ByteString as S
```

`B` 中有 lazy bytestrings 跟對應的函數，而 `S` 中則有 strict 的版本。大多數時候我們是用 lazy 的版本。

**pack** 函數的 type signature 是 `pack :: [Word8] -> ByteString`。代表他接受一串型態為 `Word8` 的 bytes，並回傳一個 `ByteString`。你能想像一個 lazy 的 list，要讓他稍微不 lazy 一些，所以讓他對於 64K lazy。

那 `Word8` 型態又是怎麼一回事？。他就像 `Int`，只是他的範圍比較小，介於 0-255 之間。他代表一個 8-bit 的數字。就像 `Int` 一樣，他是屬於 `Num` 這個 typeclass。例如我們知道 `5` 是 polymorphic 的，他能夠表現成任何數值型態。其實 `Word8` 他也能表示。

```haskell
ghci> B.pack [99,97,110]
Chunk "can" Empty
ghci> B.pack [98..120]
Chunk "bcdefghijklmnopqrstuvwx" Empty
```

正如你看到的，你其實不必特別在意 `Word8`，因為型態系統會選擇正確的型態。如果你試著用比較大的數字，像是 `336`。那對於 `Word8` 他就會變成 `80`。

我們把一些數值打包成 `ByteString`，使他們可以塞進一個 chunk 裡面。`Empty` 之於 `ByteString` 就像 `[]` 之於 list 一樣。

**unpack** 是 `pack` 的相反，他把一個 bytestring 變成一個 byte list。

**fromChunks** 接受一串 strict 的 bytestrings 並把他變成一串 lazy bytestring。**toChunks** 接受一個 lazy bytestrings 並將他變成一串 strict bytestrings。

```haskell
ghci> B.fromChunks [S.pack [40,41,42], S.pack [43,44,45], S.pack [46,47,48]]
Chunk "()*" (Chunk "+,-" (Chunk "./0" Empty))
```

如果你有很多小的 strict bytestrings 而且不想先將他們 join 起來（會耗損 memory）這樣的作法是不錯的。

bytestring 版本的 `:` 叫做 **cons**。他接受一個 byte 跟一個 bytestring，並把這個 byte 放到 bytestring 的前端。他是 lazy 的操作，即使 bytestring 的第一個 chunk 不是滿的，他也會新增一個 chunk。這也是為什麼當你要插入很多 bytes 的時候最好用 strict 版本的 `cons`，也就是 **cons'**。

```haskell
ghci> B.cons 85 $ B.pack [80,81,82,84]
Chunk "U" (Chunk "PQRT" Empty)
ghci> B.cons' 85 $ B.pack [80,81,82,84]
Chunk "UPQRT" Empty
ghci> foldr B.cons B.empty [50..60]
Chunk "2" (Chunk "3" (Chunk "4" (Chunk "5" (Chunk "6" (Chunk "7" (Chunk "8" (Chunk "9" (Chunk ":" (Chunk ";" (Chunk "<"
Empty))))))))))
ghci> foldr B.cons' B.empty [50..60]
Chunk "23456789:;<" Empty
```

你可以看到 **empty** 製造了一個空的 bytestring。也注意到 `cons` 跟 `cons'` 的差異了嗎？有了 `foldr`，我們逐步地把一串數字從右邊開始，一個個放到 bytestring 的前頭。當我們用 `cons`，我們則得到一個 byte 一個 chunk 的結果，並不是我們要的。

bytestring 模組有一大票很像 `Data.List` 中的函數。包括了 `head`，`tail`，`init`，`null`，`length`，`map`，`reverse`，`foldl`，`foldr`，`concat`，`takeWhile`，`filter`，等等。

他也有表現得跟 `System.IO` 中一樣的函數，只有 `Strings` 被換成了 `ByteString` 而已。像是 `System.IO` 中的 `readFile`，他的型態是 `readFile :: FilePath -> IO String`，而 bytestring 模組中的 **readFile** 則是 `readFile :: FilePath -> IO ByteString`。小心，如果你用了 strict bytestring 來讀取一個檔案，他會把檔案內容都讀進記憶體中。而使用 lazy bytestring，他則會讀取 chunks。

讓我們來寫一個簡單的程式，他從命令列接受兩個檔案名，然後拷貝第一個檔案內容成第二個檔案。雖然 `System.Directory` 中已經有一個函數叫 `copyFile`，但我們想要實作自己的版本。

```haskell
import System.Environment
import qualified Data.ByteString.Lazy as B

main = do
    (fileName1:fileName2:_) <- getArgs
    copyFile fileName1 fileName2

copyFile :: FilePath -> FilePath -> IO ()
copyFile source dest = do
    contents <- B.readFile source
    B.writeFile dest contents
```

我們寫了自己的函數，他接受兩個 `FilePath`（記住 `FilePath` 不過是 `String` 的同義詞。）並回傳一個 I/O action，他會用 bytestring 拷貝第一個檔案至另一個。在 `main` 函數中，我們做的只是拿到命令列引數然後呼叫那個函數來拿到一個 I/O action。

```haskell
$ runhaskell bytestringcopy.hs something.txt ../../something.txt
```

就算我們不用 bytestring 來寫，程式最後也會長得像這樣。差別在於我們會用 `B.readFile` 跟 `B.writeFile` 而不是 `readFile` 跟 `writeFile`。有很大的可能性，就是你只要 import 檔案並在函數前加上 qualified 模組名，就可以把一個用正常 String 的程式改成用 ByteString。也有可能你是要反過來做，但那也不難。

當你需要更好的效能來讀取許多資料，嘗試用 bytestring，有很大的機會你會用很小的力氣改進很多效能。我通常用正常 String 來寫程式，然後在效能不好的時候把他們改成 ByteString。

## Exceptions \(例外\)

![](img/timber.png)

所有的程式語言都有要處理失敗的情形。這就是人生。不同的語言有不同的處理方式。在 C 裡面，我們通常用非正常範圍的回傳值（像是 `-1` 或 null）來回傳錯誤。Java 跟 C\#則傾向於使用 exception 來處理失敗的情況。當一個 exception 被丟出的時候，控制流程就會跳到我們做一些清理動作的地方，做完清理後 exception 被重新丟出，這樣一些處理錯誤的程式碼可以完成他們的工作。

Haskell 有一個很棒的型態系統。Algebraic data types 允許像是 `Maybe` 或 `Either` 這種型態，我們能用這些型態來代表一些可能有或沒有的結果。在 C 裡面，在失敗的時候回傳 `-1` 是很常見的事。但他只對寫程式的人有意義。如果我們不小心，我們有可能把這些錯誤碼當作正常值來處理，便造成一些混亂。Haskell 的型態系統賦予我們更安全的環境。一個 `a -> Maybe b` 的函數指出了他會產生一個包含 `b` 的 `Just`，或是回傳 `Nothing`。這型態跟 `a -> b` 是不同的，如果我們試著將兩個函數混用，compiler 便會警告我們。

儘管有表達力夠強的型態來輔助失敗的情形，Haskell 仍然支持 exception，因為 exception 在 I/O 的 contexts 下是比較合理的。在處理 I/O 的時候會有一堆奇奇怪怪的事情發生，環境是很不能被信賴的。像是打開檔案。檔案有可能被 lock 起來，也有可能檔案被移除了，或是整個硬碟都被拔掉。所以直接跳到處理錯誤的程式碼是很合理的。

我們了解到 I/O code 會丟出 exception 是件合理的事。至於 pure code 呢？其實他也能丟出 Exception。想想看 `div` 跟 `head` 兩個案例。他們的型態是 `(Integral a) => a -> a -> a` 以及 `[a] -> a`。`Maybe` 跟 `Either` 都沒有在他們的回傳型態中，但他們都有可能失敗。`div` 有可能除以零，而 `head` 有可能你傳給他一個空的 list。

```haskell
ghci> 4 `div` 0
*** Exception: divide by zero
ghci> head []
*** Exception: Prelude.head: empty list
```

![](img/police.png)

pure code 能丟出 Exception，但 Exception 只能在 I/O section 中被接到（也就是在 `main` 的 do block 中）這是因為在 pure code 中你不知道什麼東西什麼時候會被 evaluate。因為 lazy 特性的緣故，程式沒有一個特定的執行順序，但 I/O code 有。

先前我們談過為什麼在 I/O 部份的程式要越少越好。程式的邏輯部份盡量都放在 pure 的部份，因為 pure 的特性就是他們的結果只會根據函數的參數不同而改變。當思考 pure function 的時候，你只需要考慮他回傳什麼，因為除此之外他不會有任何副作用。這會讓事情簡單許多。儘管 I/O 的部份是難以避免的（像是打開檔案之類），但最好是把 I/O 部份降到最低。Pure functions 預設是 lazy，那代表我們不知道他什麼時候會被 evaluate，不過我們也不該知道。然而，一旦 pure functions 需要丟出 Exception，他們何時被 evaluate 就很重要了。那是因為我們只有在 I/O 的部份才能接到 Exception。這很糟糕，因為我們說過希望 I/O 的部份越少越好。但如果我們不接 Exception，我們的程式就會當掉。這問題有解決辦法嗎？答案是不要在 pure code 裡面使用 Exception。利用 Haskell 的型態系統，盡量使用 `Either` 或 `Maybe` 之類的型態來表示可能失敗的計算。

這也是為什麼我們要來看看怎麼使用 I/O Excetion。I/O Exception 是當我們在 `main` 裡面跟外界溝通失敗而丟出的 Exception。例如我們嘗試打開一個檔案，結果發現他已經被刪掉或是其他狀況。來看看一個嘗試打開命令列引數所指定檔案名稱，並計算裡面有多少行的程式。

```haskell
import System.Environment
import System.IO

main = do (fileName:_) <- getArgs
            contents <- readFile fileName
            putStrLn $ "The file has " ++ show (length (lines contents)) ++ " lines!"
```

一個很簡單的程式。我們使用 `getArgs` I/O action，並綁定第一個 string 到 `fileName`。然後我們綁定檔案內容到 `contents`。最後，我們用 `lines` 來取得 line 的 list，並計算 list 的長度，並用 `show` 來轉換數字成 string。他如我們想像的工作，但當我們給的檔案名稱不存在的時候呢？

```haskell
$ runhaskell linecount.hs i_dont_exist.txt
linecount.hs: i_dont_exist.txt: openFile: does not exist (No such file or directory)
```

GHC 丟了錯誤訊息給我們，告訴我們檔案不存在。然後程式就掛掉了。假如我們希望打印出比較好一些的錯誤訊息呢？一種方式就是在打開檔案前檢查他存不存在。用 `System.Directory` 中的 **doesFileExist**。

```haskell
import System.Environment
import System.IO
import System.Directory

main = do (fileName:_) <- getArgs
            fileExists <- doesFileExist fileName
            if fileExists
                then do contents <- readFile fileName
                    putStrLn $ "The file has " ++ show (length (lines contents)) ++ " lines!"
                else do putStrLn "The file doesn't exist!"
```

由於 `doesFileExist` 的型態是 `doesFileExist :: FilePath -> IO Bool`，所以我們要寫成 `fileExists <- doesFileExist fileName`。那代表他回傳含有一個布林值告訴我們檔案存不存在的 I/O action。`doesFileExist` 是不能直接在 if expression 中使用的。

另一個解法是使用 Exception。在這個情境下使用 Exception 是沒問題的。檔案不存在這個 Exception 是在 I/O 中被丟出，所以在 I/O 中接起來也沒什麼不對。

要這樣使用 Exception，我們必須使用 `System.IO.Error` 中的 **catch** 函數。他的型態是 `catch :: IO a -> (IOError -> IO a) -> IO a`。他接受兩個參數，第一個是一個 I/O action。像是他可以接受一個打開檔案的 I/O action。第二個是 handler。如果第一個參數的 I/O action 丟出了 Exception，則他會被傳給 handler，他會決定要作些什麼。所以整個 I/O action 的結果不是如預期中做完第一個參數的 I/O action，就是 handler 處理的結果。

![](img/puppy.png)

如果你對其他語言像是 Java, Python 中 try-catch 的形式很熟，那 `catch` 其實跟他們很像。第一個參數就是其他語言中的 try block。第二個參數就是其他語言中的 catch block。其中 handler 只有在 exception 被丟出時才會被執行。

handler 接受一個 `IOError` 型態的值，他代表的是一個 I/O exception 已經發生了。他也帶有一些 exception 本身的資訊。至於這型態在語言中使如何被實作則是要看編譯器。這代表我們沒辦法用 pattern matching 的方式來檢視 `IOError`。就像我們不能用 pattern matching 來檢視 `IO something` 的內容。但我們能用一些 predicate 來檢視他們。

我們來看看一個展示 `catch` 的程式

```haskell
import System.Environment
import System.IO
import System.IO.Error

main = toTry `catch` handler

toTry :: IO ()
toTry = do (fileName:_) <- getArgs
            contents <- readFile fileName
            putStrLn $ "The file has " ++ show (length (lines contents)) ++ " lines!"

handler :: IOError -> IO ()
handler e = putStrLn "Whoops, had some trouble!"
```

首先你看到我們可以在關鍵字周圍加上 backticks 來把 `catch` 當作 infix function 用，因為他剛好接受兩個參數。這樣使用讓可讀性變好。``toTry `catch` handler`` 跟 `catch toTry handler` 是一模一樣的。`toTry` 是一個 I/O action，而 `handler` 接受一個 `IOError`，並回傳一個當 exception 發生時被執行的 I/O action。

來看看執行的結果。

```haskell
$ runhaskell count_lines.hs i_exist.txt
The file has 3 lines!

$ runhaskell count_lines.hs i_dont_exist.txt
Whoops, had some trouble!
```

在 handler 裡面我們並沒有檢查我們拿到的是什麼樣的 `IOError`，我們只是打印出 `"Whoops, had some trouble!"`。接住任何種類的 Exception 就跟其他語言一樣，在 Haskell 中也不是一個好的習慣。假如其他種類的 Exception 發生了，好比說我們送一個中斷指令，而我們沒有接到的話會發生什麼事？這就是為什麼我們要做跟其他語言一樣的事：就是檢查我們拿到的是什麼樣的 Exception。如果說是我們要的 Exception，那就做對應的處理。如果不是，我們再重新丟出 Exception。我們把我們的程式這樣修改，只接住檔案不存在的 Exception。

```haskell
import System.Environment
import System.IO
import System.IO.Error

main = toTry `catch` handler

toTry :: IO ()
toTry = do (fileName:_) <- getArgs
            contents <- readFile fileName
            putStrLn $ "The file has " ++ show (length (lines contents)) ++ " lines!"

handler :: IOError -> IO ()
handler e
    | isDoesNotExistError e = putStrLn "The file doesn't exist!"
    | otherwise = ioError e
```

除了 handler 以外其他東西都沒變，我們只接住我們想要的 I/O exception。這邊使用了 `System.IO.Error` 中的函數 **isDoesNotExistError** 跟 **ioError**。`isDoesNotExistError` 是一個運作在 `IOError` 上的 predicate ，他代表他接受一個 `IOError` 然後回傳 `True` 或 `False`，他的型態是 `isDoesNotExistError :: IOError -> Bool`。我們用他來判斷是否這個錯誤是檔案不存在所造成的。我們這邊使用 guard，但其實也可以用 if else。如果 exception 不是由於檔案不存在所造成的，我們就用 `ioEroror` 重新丟出接到的 exception。他的型態是 `ioError :: IOException -> IO a`，所以他接受一個 `IOError` 然後產生一個會丟出 exception 的 I/O action。那個 I/O action 的型態是 `IO a`，但他其實不會產生任何結果，所以他可以被當作是 `IO anything`。

所以有可能在 `toTry` 裡面丟出的 exception 並不是檔案不存在造成的，而 ``toTry `catch` handler`` 會接住再丟出來，很酷吧。

程式裡面有好幾個運作在 `IOError` 上的 I/O action，當其中一個沒有被 evaluate 成 `True` 時，就會掉到下一個 guard。這些 predicate 分別為：

* **isAlreadyExistsError**
* **isDoesNotExistError**
* **isFullError**
* **isEOFError**
* **isIllegalOperation**
* **isPermissionError**
* **isUserError**

大部分的意思都是顯而易見的。當我們用了 **userError** 來丟出 exception 的時候，`isUserError` 被 evaluate 成 `True`。例如說，你可以寫 `ioError $ userError "remote computer unplugged!"`，儘管用 `Either` 或 `Maybe` 來表示可能的錯誤會比自己丟出 exception 更好。

所以你可能寫一個像這樣的 handler

```haskell
handler :: IOError -> IO ()
handler e
    | isDoesNotExistError e = putStrLn "The file doesn't exist!"
    | isFullError e = freeSomeSpace
    | isIllegalOperation e = notifyCops
    | otherwise = ioError e
```

其中 `notifyCops` 跟 `freeSomeSpace` 是一些你定義的 I/O action。如果 exception 不是你要的，記得要把他們重新丟出，不然你的程式可能只會安靜地當掉。

`System.IO.Error` 也提供了一些能詢問 exception 性質的函數，像是哪些 handle 造成錯誤，或哪些檔案名造成錯誤。這些函數都是 `ioe` 當開頭。而且你可以在文件中看到一整串詳細資料。假設我們想要打印出造成錯誤的檔案名。我們不能直接打印出從 `getArgs` 那邊拿到的 `fileName`，因為只有 `IOError` 被傳進 handler 中，而 handler 並不知道其他事情。一個函數只依賴於他所被呼叫時的參數。這也是為什麼我們會用 **ioeGetFileName** 這函數，他的型態是 `ioeGetFileName :: IOError -> Maybe FilePath`。他接受一個 `IOError` 並回傳一個 `FilePath`（他是 `String` 的同義詞。）基本上他做的事就是從 `IOError` 中抽出檔案路徑。我們來修改一下我們的程式。

```haskell
import System.Environment
import System.IO
import System.IO.Error

main = toTry `catch` handler

toTry :: IO ()
toTry = do (fileName:_) <- getArgs
    contents <- readFile fileName
    putStrLn $ "The file has " ++ show (length (lines contents)) ++ " lines!"

handler :: IOError -> IO ()
handler e
    | isDoesNotExistError e =
        case ioeGetFileName e of Just path -> putStrLn $ "Whoops! File does not exist at: " ++ path
                                 Nothing -> putStrLn "Whoops! File does not exist at unknown location!"
    | otherwise = ioError e
```

在 `isDoesNotExistError` 是 `True` 的 guard 裡面，我們在 case expression 中用 `e` 來呼叫 `ioeGetFileName`，然後用 pattern matching 拆出 `Maybe` 中的值。當你想要用 pattern matching 卻又不想要寫一個新的函數的時候，case expression 是你的好朋友。

你不想只用一個 `catch` 來接你 I/O part 中的所有 exception。你可以只在特定地方用 `catch` 接 exception，或你可以用不同的 handler。像這樣：

```haskell
main = do toTry `catch` handler1
          thenTryThis `catch` handler2
          launchRockets
```

這邊 `toTry` 使用 `handler1` 當作 handler，而 `thenTryThis` 用了 `handler2`。`launchRockets` 並不是 `catch` 的參數，所以如果有任何一個 exception 被丟出都會讓我們的程式當掉，除非 `launchRockets` 使用 `catch` 來處理 exception。當然 `toTry`，`thenTryThis` 跟 `launchRockets` 都是 I/O actions，而且被 do syntax 綁在一起。這很像其他語言中的 try-catch blocks，你可以把一小段程式用 try-catch 包住，你可以自己調整該包多少進去。

現在你知道如何處理 I/O exception 了。我們並沒有提到如何從 pure code 中丟出 exception，這是因為正如我們先前提到的，Haskell 提供了更好的辦法來處理錯誤。就算是在可能會失敗的 I/O action 中，我也傾向用 `IO (Either a b)`，代表他們是 I/O action，但當他們被執行，他們結果的型態是 `Either a b`，意思是不是 `Left a` 就是 `Right b`。

