# 設計製作論実習3

## 第3回

千葉工業大学 上田 隆一

<br />

<p style="font-size:50%">
This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons Attribution-ShareAlike 4.0 International License</a>.
<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">
<img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a>
</p>

---

### 連絡事項

* WSLでロボットに直接接続するのは難しそう
    * ラズパイ経由でも実験中
    * Linuxネイティブな環境を準備するのが良さそう
    * 仮想マシンでもたぶんいけるけど重たいかも

---

### 今後の予定

* 第1,2週: セットアップ
* 第3週（10/7）: シミュレータ環境の使い方
* 第4週（10/14）: 実機を使う方法
* 5〜9週: 何か簡単なものを作る
  * 第5週（10/21）: なにやるか考える
  * 第6週（10/28）: プレゼン
  * 第7週（11/4）: 作業
  * 第8週（11/11）: 作業
  * 第9週（11/18）: 発表
* 10〜15週: 5〜9週のように何か作る


---

### 「何か作る」

* ちゃんと作品を残す
    * 作り方のドキュメントがインターネット上に公開される
    * GitHubにコードが整理されて置かれる
    * YouTubeにデモがある

---

### カメラを扱う

* シミュレータ（Gazebo）
* 実機

---

### Gazebo環境内でのカメラの利用

---

### カメラを定義したファイルのリンク追加

* 手順
    * `crane_x7_ros/crane_x7_description/urdf/crane_x7.xacro`ファイルに次のように1行追加
        * urdfファイル: ロボットのリンクや関節を記述したファイル 
        * xacroファイル: urdfファイルを簡潔に書いたもの

```
<?xml version="1.0"?>
<robot xmlns:xacro="http://ros.org/wiki/xacro">
・・・
  <xacro:include filename="$(find crane_x7_description)/urdf/crane_x7_wide_two_finger_gripper.xacro"/>
  <!--追加！！！-->
  <xacro:include filename="$(find crane_x7_description)/urdf/camera.urdf"/>
・・・
```

---

### カメラの形状の記述（1/3）

* `crane_x7_ros/crane_x7_description/urdf/camera.urdf`の記述
    * XMLで以下を記述
        * リンク（棒）を一本
        * ロボットとリンクを取り付ける固定関節を一個

```
<?xml version="1.0"?>

<robot name="camera">

  <link name="camera_link">
     ・・・リンクに関する記述・・・
  </link>

  <joint name="camera_joint" type="fixed">
     ・・・関節に関する記述・・・
  </joint>

</robot>
```

---

### カメラの形状の記述（2/3）

* リンクの記述
    * inertia: 慣性モーメント（この例では適当に軽く設定してある）
    * visual: カメラの形状（1x5x3[cm]の箱）

```
<link name="camera_link">
  <inertial>
    <mass value="1e-6"/>
    <origin xyz="0 0 0" rpy="0 0 0"/>
    <inertia ixx="1.0" ixy="0.0" ixz="0.0" iyy="1.0" iyz="0.0" izz="1.0"/>
  </inertial>

  <visual>
    <geometry>
      <box size="0.01 0.05 0.03"/>
    </geometry>
    <material name="black">
      <color rgba="0 0 0 1" />
    </material>
  </visual>
</link>
```

---

### カメラの形状の記述（3/3）

* 関節の記述
    * `parent, child`に関節の両側のリンクを指定
    * `origin`が関節の位置（`parent`基準）
        * `rpy`はロール・ピッチ・ヨー
    * この定義の場合、カメラの箱が宙に浮く

```
<joint name="camera_joint" type="fixed">
  <parent link="crane_x7_gripper_base_link"/>
  <child  link="camera_link"/>
  <origin xyz="0 0.1 0" rpy="0 1.570796326795 0"/>
</joint>
```

---

### Gazebo上での確認

* 手首の横に箱が出て、アームを動かすと一緒に動く
    * 箱がロボットにめり込んでいるとMoveIt!は動かないので注意

```
（Gazeboのロボットを立ち上げておく）
$ rosrun crane_x7_examples pose_groupstate_example.py 
```

<img width="65%" src="./figs/camera_attached_robot.png" />

---

### カメラの機能の記述

* 作った箱をGazeboの世界を覗くためのカメラにする
    * [ここにあるサンプル](http://gazebosim.org/tutorials?tut=ros_gzplugins#Camera)をコピペ
        * `gazebo`要素を`camera.urdf`の`robot`要素の中に追加
        * リンクの名前を`camera_link`、カメラの名前を`camera1`にしておく

```
<gazebo reference="camera_link">
  <sensor type="camera" name="camera1">
    <update_rate>30.0</update_rate>
       ・・・
    <plugin name="camera_controller" filename="libgazebo_ros_camera.so">
      <alwaysOn>true</alwaysOn>
      <updateRate>0.0</updateRate>
      <cameraName>camera1</cameraName>
      <imageTopicName>image_raw</imageTopicName>
      <cameraInfoTopicName>camera_info</cameraInfoTopicName>
      <frameName>camera_link</frameName>
       ・・・
    </plugin>
  </sensor>
</gazebo>
```

---

### 動作確認（画像を見る手順）

```
$ sudo apt install ros-melodic-image-view
（`source`等が必要）
（Gazeboのロボットを立ち上げておく）
$ rosrun image_view image_view image:=/camera1/image_raw
```

<img width="45%" src="./figs/camera_image.png" />

* 本当はカメラの向きをハンドの向きと合わせておいたほうが良い
    * 各自おまかせします

---

### 動作確認（ロボットを動かす）


<video controls src="./figs/robot_camera.mov" />
