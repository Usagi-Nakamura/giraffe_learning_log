# giraffe_learning_log
Giraffeの独習記録のためのリポジトリです。
## 2022年５月７日までの取組
GiraffeのGithubのDocumentはとても詳しく書いてありますが、やみくもに読んでも、なかなか理解できませんでした。  

しばらく読んで投げ出しかけたとき、とりあえずDemoを動かしてみて、そのコードを少しずつ変えながら動きがどのように変わるかを見ることとし、それがわからないときにGithubのドキュメントに戻ってみては、と思い立ちました。  

そうしたところ少し糸口が見えてきたような気がしました。  

まずは、コマンドプロンプトから

    dotnet new giraffe
    
しました。出てきたコードは、コードについているコメントによると次の大きなブロックに分かれるようです。

1. model
1. view
1. handlerの定義
1. webApp (routing)
1. error handler
1. config and main  

このうち、giraffe本家のドキュメントにmain building blockといわれている`httpHandler`のあるhandlerの定義のところから実験を始めることとしました。

### httpHandlerをいじっていみる

まずは、`webApp` にある、routingのpathのパラメータを次のように１つ増やしてみました。

    routef "/hello/%s/%i" indexHandler  

そうすると`routef`の第二パラメータの`indexHandler`に波線がつきますが、ホバーして内容を見るとindexHandlerの型が一致しません、と出ます。  
routefの第一パラメータのstringの内容を`"/hello/%s/%i"`を変えただけで、第二パラメータの型が変化するのは驚きましたが、この理由は今後の調査項目にするとして、とりあえずこの第二パラメータの型に合わせて`indexHandler`の定義を次のように変えてみます。  

    let indexHandler  (name: string, times_size : int) =  
        let greetings =   
        sprintf "Hello %s, from Giraffe!" (name.PadLeft(times_size, '#'))  
        let model     = { Text = greetings }  
        let view      = Views.index model  
        htmlView view

これで無事、パラメータを変化させたものが想定通り動くことが確認できました。

### httpHandlerを新しく作ってみる

また、もう一つ Handler を作ってみました。先ほど同様、`webApp` で

    routef "/another/%s" anotherHandler

を追加し、anotherHandlerを次のように定義します。

    let anotherHandler (name: string)  =  
        let greeting = sprintf "Howdy %s" name  
        let model = { Text = greeting }  
        let view = Views.index model  
        htmlView view  

こちらも問題なく動くことが確認できました。
### 引き続きViewをいじってみる

今度はViewの部分をいじってみます。次の２つの関数を追加してみます。

    let partialEx () =
        h1 [] [ str "another view!!" ]

    let another_view (model: Message) =
        [
            partialEx()
            p [] [ str model.Text ]
        ] |> layout

この`another_view`を先ほど作った`anotherHandler`に次のように適用してみます。

    let anotherHandler (name: string)  =
        let greeting = sprintf "Howdy %s" name
        let model = { Text = greeting }
        let view = Views.another_view model
        htmlView view

これで無事Viewも作れることがわかりました。次はmodelを新しく作ってみようと思います。

### modelを追加してみる（ 2022年５月19日 ）

hanlderとviewを新しく追加できましたので次は予定通りmodelを追加してみました。  
まずは、modelのセクションで、Messageのrecord定義があるところで次の定義を追加しました。

    type AnotherMessage =
        {
            _data : string list
        }

その後、以前追加した次のviewのパラメータを上で作った定義に次のように差し替えます。

    let another_view (model : AnotherMessage) =
        [
            partialEx()
            p [] [ li [] [str (model._data.Item 3) ] ]
        ] |> layout

最後にこれを扱うハンドラも次のように差し替えます。

    let anotherHandler (name: string)  =
        let greeting = sprintf "Howdy %s" name
        let model = 
            { _data = ["abc"; "def"; "ghi"; "jkl"; "mno"] }
        let view = Views.another_view model
        htmlView view

これで無事に想定通りの動きになりました。
