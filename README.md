# PSRの概要、クラスとインターフェイス

以前にミーティングでちらっと話しましたが、PHP-FIGとPSRの存在をご存知でしょうか？  

PHP-FIGとは、フレームワークやライブラリの垣根を超えた、PHPの開発者団体です。  
PSRはPHP-FIGが策定している、基本的な規約とインターフェイスをまとめたものものです。

[PHP-FIG公式](https://www.php-fig.org/)  
[PHP-FIG構成メンバー](https://www.php-fig.org/personnel/)  
[PSR一覧](https://www.php-fig.org/psr/)  

この10年ほどでPHPの世界は、PSRなしに語れないものになっているようです。  
複数の有名なフレームワークの開発者たちが、PHP-FIGから離脱したにもかかわらず、その流れは止まりませんでした。  
いまでは様々なライブラリ、フレームワークがPSRに準拠し、相互に運用可能なエコシステムを構築しつつあります。  

注意したいのは、PSRはあくまでも勧告であり、決して従う義務があるものではないということです。  
しかし、PSRを中心とした世界観へシフトする流れは、すでにPHPコミュニティの主流となりつつあり、  
私達が次の20年もPHPを利用し、その恩恵に与ろうとするのであれば、PSRを知ることは欠かせないと考えます。  

## ライブラリの相互運用

たとえば、HTTPメッセージを扱ういくつかのライブラリは、PSR-7という同じInterfaceを実装しているため、容易に取り替え可能です。  
利用しているライブラリが目的に沿わなくなったり、メンテナンスされなくなってしまったとしても、  
同じInterfaceを実装したライブラリが他にあれば、移行コストを小さく抑えやすくなります。

[オブジェクト インターフェイス](https://www.php.net/manual/ja/language.oop5.interfaces.php)
> 同じインターフェイスを実装していることで、 開発者が交換可能な異なるクラスを作成できるようにします。 同じインターフェイスを持つクラスによくある例として、 複数のデータベースにアクセスするサービスや 決済のゲートウェイ、 異なるキャッシュ戦略が挙げられます。 実装が異なっていても、 それを使うコードに変更を加えることなく、それらを交換することができます。  

私も以前、PSR-11に沿った簡易的なライブラリを作成し、OSSのSlimフレームワークに組み込んでみました。  
SlimがPSR-11に準拠した実装を要求している、ということさえ知っていれば、その内部に関心を払うことなく、  
求められた実装を渡してあげるだけで、Slimフレームワークのモジュールとして機能させることができるのです。  

## クラスとインターフェイスの例

```PHP
// インターフェイス
interface HumanInterface {
    public function getName(): string;
}

// クラス
class Male implements HumanInterface {
    private $name = '男性の名前';

    public function getName(): string {
        return $this->name;
    }
}

// クラス
class Female implements HumanInterface {
    private $name = '女性の名前';

    public function getName(): string {
        return $this->name;
    }
}

/* 
HumanInterfaceにgetNameメソッドが定義されているので、
HumanInterfaceを実装するクラスは、同じ定義でgetNameメソッドを実装しなければエラーになる。
*/

// インスタンス化
$male = new Male();
$female = new Female();

// 同じオブジェクトかどうか
var_dump($male === $female); // false

// それぞれのオブジェクトが、HumanInterfaceのインスタンスかどうか
var_dump($male instanceof HumanInterface); // true
var_dump($female instanceof HumanInterface); // true

// 引数の型としてHumanInterfaceを指定した無名関数
$echoHumanName = function (HumanInterface $human): bool {
    echo $human->name;
};

// 引数にオブジェクトを渡して実行
$echoHumanName($male); // 男性の名前
$echoHumanName($female); // 女性の名前
```

## SlimとPSR-11

Slimはこれまでにも紹介しているとおり、PHPのマイクロフレームワークです。  
いくつかのPSRに準拠した構成になっています。

PSR-11はDIコンテナのインターフェイスです。  
DIコンテナが何であるかという説明は省きます。  

以下はPSR-11に準拠することでライブラリを相互運用可能であることの簡単な例示です。  

```PHP
use \Knp\DiContainer\DiContainer; // 自作のDIコンテナライブラリ
use \League\Container\Container; // The PHP Leagueという開発者団体が作ったDIコンテナライブラリ
use \Psr\Container\ContainerInterface; // PSR-11のインターフェイス
use \Slim\Factory\AppFactory; // Slimのファクトリークラス

// theleaguephp/containerをインスタンス化
$league_container = new Container();

// 自作のDIコンテナライブラリをインスタンス化
$knp_container = new DiContainer();

// それぞれのオブジェクトが、ContainerInterfaceを実装しているかどうか
var_dump($league_container instanceof ContainerInterface); // true
var_dump($knp_container instanceof ContainerInterface); // true

// DIコンテナをSlimに登録
// 同じContainerInterfaceを実装しているので、どちらを渡しても動作する
AppFactory::setContainer($league_container);
AppFactory::setContainer($knp_container);
```
