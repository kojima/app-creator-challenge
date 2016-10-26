# ゲーム完成イメージ

<img src="./drop-game-sh.png" width="360" />

画面をタップしてバケツを移動し、落ちてくる雨をキャッチするゲーム！

参考元: <a href="https://github.com/libgdx/libgdx/wiki/A-simple-game">A simple game (ligGDX)</a>

## 準備
1. Xcode(バージョン8以上)をインストール
2. <a href="./assets.zip">ゲームアセット(assets.zip)</a>をダウンロードして展開する

## SpriteKitプロジェクトを作成
1. Xcodeを起動し、「Create a new Xcode project」をクリック(画像参照)<br/>
<img src="create-a-new-project.png" height="240" />
2. 作成するプロジェクトのテンプレートとして、「Game」を選択(画像参照)<br/>
<img src="select-the-game-template.png" height="240" />
3. プロジェクトの情報として、以下を入力する:
 * Product name: __drop__
 * Team: <span style="color: #e74c3c;">適宜選択</span>
 * Organization Name: <span style="color: #e74c3c;">適宜選択 (例: 氏名等)</span>
 * Organization Identifier: __com.example__
 * Language: __Swift__
 * Game Category: __SpriteKit__
 * Devices: __iPhone__
 * Integrate GameplayKit: チェックを外す
 * Include Unit Tests: チェックを外す
 * Include UI Tests: チェックを外す
4. 最後にプロジェクトの保存先を指定して、「Create」ボタンをクリックしてプロジェクトを作成

## 実機での実行確認
1. iPhoneをLightningケーブルでMacに接続する
2. 実行ボタンをクリック
3. 接続したiPhoneでサンプルのゲームが実行できることを確認する<br/>
(画像参照)<br/>
<img src="run-the-example-app.png" height="240" />

## 画面表示方向の設定
下図に従って、画面の表示方向を横長(Landscape Left/Landscape Right)のみにする:<br/>
<img src="device-orientation.png" height="240" />

## `GameViewController`クラスの設定
1. プロジェクトナビゲーター中の__GameViewController.swift__をクリック(画像参照)<br/>
<img src="navigator-gameviewcontroller.png" height="240" />
2. 右側のソースコードエディタで、`viewDidLoad`メソッド中の`scene.scaleMode = .aspectFill`を`scene.scaleMode = .resizeFill`に書き換える:

```swift
override func viewDidLoad() {
    super.viewDidLoad()

    if let view = self.view as! SKView? {
        // Load the SKScene from 'GameScene.sks'
        if let scene = SKScene(fileNamed: "GameScene") {
            // Set the scale mode to scale to fit the window
            scene.scaleMode = .resizeFill

            // Present the scene
            view.presentScene(scene)
        }

        view.ignoresSiblingOrder = true

        view.showsFPS = true
        view.showsNodeCount = true
    }
}
```

## ゲームアセットの追加
1. プロジェクトナビゲーター中の__Assets.xcassets__をクリック
2. 「準備」の2でダウンロードして展開しておいたフォルダから__bucket.png__と__droplet.png__を選択し、Assets.xcassetsにドラッグ・アンド・ドロップしてゲームアセットとして追加する(画像参照)<br/>
<img src="add-game-assets.png" height="240" />

## `GameScene`クラスの変更 (ゲームコードの実装)

``` swift
//
//  GameScene.swift
//  drop
//
//  Created by Hidenori Kojima on 2016/10/25.
//  Copyright © 2016年 Hidenori Kojima. All rights reserved.
//

import SpriteKit
import GameplayKit

class GameScene: SKScene {

    private var label : SKLabelNode?
    private var spinnyNode : SKShapeNode?

    override func didMove(to view: SKView) {

        // Get label node from scene and store it for use later
        self.label = self.childNode(withName: "//helloLabel") as? SKLabelNode
        if let label = self.label {
            label.alpha = 0.0
            label.run(SKAction.fadeIn(withDuration: 2.0))
        }

        // Create shape node to use during mouse interaction
        let w = (self.size.width + self.size.height) * 0.05
        self.spinnyNode = SKShapeNode.init(rectOf: CGSize.init(width: w, height: w), cornerRadius: w * 0.3)

        if let spinnyNode = self.spinnyNode {
            spinnyNode.lineWidth = 2.5

            spinnyNode.run(SKAction.repeatForever(SKAction.rotate(byAngle: CGFloat(M_PI), duration: 1)))
            spinnyNode.run(SKAction.sequence([SKAction.wait(forDuration: 0.5),
                                              SKAction.fadeOut(withDuration: 0.5),
                                              SKAction.removeFromParent()]))
        }
    }


    func touchDown(atPoint pos : CGPoint) {
        if let n = self.spinnyNode?.copy() as! SKShapeNode? {
            n.position = pos
            n.strokeColor = SKColor.green
            self.addChild(n)
        }
    }

    func touchMoved(toPoint pos : CGPoint) {
        if let n = self.spinnyNode?.copy() as! SKShapeNode? {
            n.position = pos
            n.strokeColor = SKColor.blue
            self.addChild(n)
        }
    }

    func touchUp(atPoint pos : CGPoint) {
        if let n = self.spinnyNode?.copy() as! SKShapeNode? {
            n.position = pos
            n.strokeColor = SKColor.red
            self.addChild(n)
        }
    }

    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        if let label = self.label {
            label.run(SKAction.init(named: "Pulse")!, withKey: "fadeInOut")
        }

        for t in touches { self.touchDown(atPoint: t.location(in: self)) }
    }

    override func touchesMoved(_ touches: Set<UITouch>, with event: UIEvent?) {
        for t in touches { self.touchMoved(toPoint: t.location(in: self)) }
    }

    override func touchesEnded(_ touches: Set<UITouch>, with event: UIEvent?) {
        for t in touches { self.touchUp(atPoint: t.location(in: self)) }
    }

    override func touchesCancelled(_ touches: Set<UITouch>, with event: UIEvent?) {
        for t in touches { self.touchUp(atPoint: t.location(in: self)) }
    }


    override func update(_ currentTime: TimeInterval) {
        // Called before each frame is rendered
    }
}
```

``` swift
//
//  GameScene.swift
//  drop
//
//  Created by Hidenori Kojima on 2016/10/25.
//  Copyright © 2016年 Hidenori Kojima. All rights reserved.
//

import SpriteKit
import GameplayKit

class GameScene: SKScene {

    private var label : SKLabelNode?
    private var spinnyNode : SKShapeNode?

    override func didMove(to view: SKView) {
    }

    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
    }

    override func update(_ currentTime: TimeInterval) {
        // Called before each frame is rendered
    }
}
```

## `GameViewController.swift`
``` swift
//
//  GameViewController.swift
//  drop
//
//  Created by Hidenori Kojima on 2016/10/25.
//  Copyright © 2016年 Hidenori Kojima. All rights reserved.
//

import UIKit
import SpriteKit
import GameplayKit

class GameViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()

        if let view = self.view as! SKView? {
            // Load the SKScene from 'GameScene.sks'
            if let scene = SKScene(fileNamed: "GameScene") {
                // Set the scale mode to scale to fit the window
                scene.scaleMode = .resizeFill

                // Present the scene
                view.presentScene(scene)
            }

            view.ignoresSiblingOrder = true

            view.showsFPS = true
            view.showsNodeCount = true
        }
    }

    override var shouldAutorotate: Bool {
        return true
    }

    override var supportedInterfaceOrientations: UIInterfaceOrientationMask {
        if UIDevice.current.userInterfaceIdiom == .phone {
            return .allButUpsideDown
        } else {
            return .all
        }
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Release any cached data, images, etc that aren't in use.
    }

    override var prefersStatusBarHidden: Bool {
        return true
    }
}
```

## `GameScene.swift`
``` swift
//
//  GameScene.swift
//  drop
//
//  Created by Hidenori Kojima on 2016/10/25.
//  Copyright © 2016年 Hidenori Kojima. All rights reserved.
//

import SpriteKit
import GameplayKit

class GameScene: SKScene, SKPhysicsContactDelegate {

    private let dropletCategory: UInt32 = 0x1 << 1
    private let bucketCategory: UInt32 = 0x1 << 0

    private var droplet: SKSpriteNode!
    private var bucket : SKSpriteNode!

    override func didMove(to view: SKView) {

        physicsWorld.contactDelegate = self

        droplet = SKSpriteNode(imageNamed: "droplet")
        droplet.zPosition = 10
        droplet.physicsBody = SKPhysicsBody(rectangleOf: CGSize(width: 64.0, height: 64.0))
        droplet.physicsBody?.affectedByGravity = false
        droplet.physicsBody?.categoryBitMask = dropletCategory
        droplet.physicsBody?.contactTestBitMask = bucketCategory
        droplet.physicsBody?.collisionBitMask = 0

        bucket = SKSpriteNode(imageNamed: "bucket")
        bucket.physicsBody = SKPhysicsBody(rectangleOf: CGSize(width: 64.0, height: 64.0))
        bucket.physicsBody?.mass = 0
        bucket.physicsBody?.affectedByGravity = false
        bucket.physicsBody?.categoryBitMask = bucketCategory
        bucket.physicsBody?.contactTestBitMask = dropletCategory
        bucket.physicsBody?.collisionBitMask = 0
        bucket.position.x = 0
        bucket.position.y = -size.height * 0.5 + 40
        addChild(bucket)

        run(SKAction.repeatForever(
                SKAction.sequence([
                    SKAction.run { self.spawnRaindrop() },
                    SKAction.wait(forDuration: 1.0)
                ])))

        run(SKAction.repeatForever(SKAction.playSoundFileNamed("rain.mp3", waitForCompletion: true)))
    }

    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        if let touch = touches.first {
            let position = touch.location(in: self)
            bucket.position.x = position.x
        }
    }

    override func update(_ currentTime: TimeInterval) {
        if bucket.position.x < -size.width * 0.5 {
            bucket.position.x = -size.width * 0.5
        }
        if bucket.position.x > size.width * 0.5 {
            bucket.position.x = size.width * 0.5
        }
    }

    func didBegin(_ contact: SKPhysicsContact) {
        var droplet: SKNode!
        if contact.bodyA.categoryBitMask == dropletCategory {
            droplet = contact.bodyA.node!
        } else if contact.bodyB.categoryBitMask == dropletCategory {
            droplet = contact.bodyB.node!
        }
        if droplet != nil {
            droplet.run(SKAction.sequence([
                            SKAction.playSoundFileNamed("drop.wav", waitForCompletion: false),
                            SKAction.removeFromParent()
                        ]))
        }
    }

    private func spawnRaindrop() {
        if let dl = droplet.copy() as? SKSpriteNode {
            let x = CGFloat(arc4random_uniform(UInt32(size.width))) - size.width * 0.5
            dl.position.x = x
            dl.position.y = size.height * 0.5 + 32
            dl.run(SKAction.sequence([SKAction.moveBy(x:  0, y: -size.height - 64, duration: 2.0),
                                      SKAction.removeFromParent()]))
            addChild(dl)
        }
    }
}
```

``` swift
override func update(_ currentTime: TimeInterval) {
    if bucket.position.x < -size.width * 0.5 {
        bucket.position.x = -size.width * 0.5
    }
    if bucket.position.x > size.width * 0.5 {
        bucket.position.x = size.width * 0.5
    }
    if let data = motionManager.accelerometerData {
        if fabs(data.acceleration.y) > 0.2 {
            if UIApplication.shared.statusBarOrientation == .landscapeLeft {
                bucket.position.x += 5 * (data.acceleration.y > 0 ? 1: -1)
            } else {
                bucket.position.x -= 5 * (data.acceleration.y > 0 ? 1: -1)
            }
        }
    }
}
```


## ゲームアセット
* 雨粒画像
 * https://www.box.com/s/peqrdkwjl6guhpm48nit
* バケツ画像
 * https://www.box.com/s/605bvdlwuqubtutbyf4x
* 水滴が落ちる音 ([CC 3.0](https://creativecommons.org/licenses/by/3.0/))
 * http://www.freesound.org/people/junggle/sounds/30341/
* 雨の音 ([CC 3.0](https://creativecommons.org/licenses/by/3.0/))
 * http://www.freesound.org/people/acclivity/sounds/28283/
