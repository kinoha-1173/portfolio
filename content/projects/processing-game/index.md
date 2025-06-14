+++
date = '2025-06-13T15:52:36+09:00'
draft = false
title = 'Processingで通信ゲーム【縛りあり】'
thumbnail = 'thumbnail.png'
summary = 'Processing標準ライブラリ+Processing.netのみで通信ゲームを制作しました。Javaに慣れた中でのProcessingは縛りの塊でした。'
+++
## 制作内容
### 成果物
- Processingによる通信ゲーム

Processnigのソケット通信機能を使ったプログラムを作成しました。  
ホスト機が複数端末の接続を受け付け、各クライアントからのデータを送受信します。クライアントはサーバへデータを送信しつつ、サーバからのデータを基に描画処理を行います。

{{<video src="2024-11-18_9.36.58.mov" fallback="お使いのブラウザでは動画の再生ができません">}}

### 制作期間
- １週間

実際にキーボードを弾いた時間は数時間だと思いますが、制作委任〜納品の期間を考えると、制作全体として１週間程度かと思います。

## 使用技術
- Processing
  - 標準ライブラリ  
  [公式レファレンス](https://processing.org/reference/)に示されているクラス・メソッド
  - Processing.netライブラリ  
  [Processing.net](https://processing.org/reference/libraries/net/index.html)に示されているライブラリメソッド  
  ソケット通信のために利用

### 縛り
- 上記に示したライブラリのみを使用すること  
標準・ネットワークライブラリのみを利用します。他の外部ライブラリやJavaライブラリはプログラム作成に利用しません。

## 動機
### 学部内専攻体験会
学部内の有志団体で１年次生に向けた専攻体験会が企画され、そのうち私が所属する専攻の企画を委任されました。

私が所属するNS：ネットワークシステムプログラムでは、２年次前期の必修科目内でJavaによるソケット通信を利用したゲーム作成が最終課題でした。  
そこで、NSを選んでちゃんと習得すると、こんなプログラムを作る力が付くということを伝えるために、このプロダクトを企画しました。

## その他説明
### なぜProcessing？
**１年次生が確実に使える環境**だったからです。

弊学部の１年次必修科目では、Processingを利用するものがあります。一応、後期の選択科目にJavaを利用するものもあります。  
企画の開催時期は後期半ば。使おうと思えばJavaも使えたのですが、体験会に来る１年次生全員がJavaのIDEをインストールしているとは限りません。このためだけにインストールするのは容量の無駄ですし、時間も使ってしまいます。  
そのため、参加者全員が、参加時点で必ず利用できるような環境としてProcessingを選択しました。

### なぜ縛り？
**参加者のレベルに少しでも合わせる**ことを目指したためです。

当時の１年次生に対して、Processingのレファレンスの存在は知っている(けど見ながら作る習慣はない)くらいのレベルを想定していました。  
体験内容としてコードの穴埋めを企画したのですが、そこで「公式レファレンスに載っていない内容」の実装をさせるのは、参加者のレベルに見合わないと考えました。

正直なところ、なかなか辛かったです。そもそも私がProcessingの考え方をあまり好いていなかったことに加え、Processing自体は標準入力機能を備えていません。これが実装の自由度を下げてくれました。  
最終的には「ボタン押下状況をサーバに毎フレーム送信する」という非常にパワーな実装となりました。

ちなみに、企画内に参加者に難しさを聞いたところ、そこそこ難しかったとのことです。人のコードを読みつつ、自分で埋めること自体、難しいですからね…

## 展望・改善点
### 脳筋→スマートに
今回の実装は、毎フレームデータの送受信を行い、クライアントはさらに描画処理も行います。しかも、クライアントの描画コストは結構大きくなっています。  
毎フレームデータを送受信するのは不正な処理にも繋がり、最終的にフレームレートを落として処置するという暴挙にも出ています。

```java
//Input List that controlles my-Tank's moving
int[] movement = {0, 0, 0, 0, 0, 0, 0}; //[w, s, a, d, l, h, space or enter]

void draw(){
  background(255);
  //create send message
  send = myNo + " " + movement[0] + " " + movement[1] + " " + movement[2] +
         " " + movement[3] + " " + movement[4] + " " + movement[5] + " " + movement[6];
  myClient.write(send);
  ...
}
```

せめて、クライアントはボタン押下状況が変わったときのみデータ送信を行うとか、それに応じてサーバの処理を行うとかをすべきだと思います。

### 処理をサーバサイドへ
今回の実装では、描画に使うクライアントの座標データを各クライアントが持っています。さらに問題なのは、座標の移動処理も各クライアントが行っています。  
つまり、サーバはボタン押下状況と接続状況を送受信しているだけなのです。このままでは、各クライアント内で座標データの不整合が起こる可能性があります。それはゲームとしては非常にイケないことです。低評価、星１待ったなしです。

```java
void draw(){
    ...
    if(myClient.available() > 0){
        received = split(myClient.readString(), ' '); myClient.clear();
        println(received); myClient.clear();
        speed[0] = int(received[1]); speed[1] = int(received[2]); speed[2] = int(received[3]);
        speed[3] = int(received[4]); speed[4] = int(received[5]); speed[5] = int(received[6]);
        speed[6] = int(received[7]);
        TankList.get(received[0]).drawTank(speed);
    }
}

class Tank{
    //drawing client's character
    void drawTank(){
        if(me){
        stroke(0, 255, 0); fill(0, 255, 0);
        } else {
        stroke(127); fill(127);
        }
        circle(x, y, 50);
        pushMatrix();
        ...
    }

    //calcrate client's point
    void drawTank(int[] movement){
        xx = movement[3] - movement[2];
        yy = movement[1] - movement[0];
        aa = movement[5] - movement[4];
        if(xx != 0 || yy != 0) moveTank(xx, yy);
        if(aa != 0) rotateTank(aa);
        drawTank();
    }
}
```

そのため、座標処理もサーバで行うべきだと考えています。それに対応するには、サーバ上に各クライアントのデータを置くのが適切だとも思います。  
今回の制作はけっこう急ぎの案件だったとはいえ、さすがに脳筋が過ぎたと感じています。これは反省点です。