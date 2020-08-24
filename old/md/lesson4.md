# 設計製作論実習3

## 第4回

千葉工業大学 上田 隆一

<br />

<p style="font-size:50%">
This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons Attribution-ShareAlike 4.0 International License</a>.
<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">
<img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a>
</p>

---

### 立ち上げ

* PCにロボットをUSBで接続して次のコマンドを実行
    * うまくいくとゆっくりマニピュレータが直立
```
$ sudo chmod 777 /dev/ttyUSB0 
$ roslaunch crane_x7_control crane_x7_control.launch
```
    * アームを動かす
```
$ roslaunch crane_x7_bringup demo.launch fake_execution:=false
別の端末で$ rosrun crane_x7_examples pose_groupstate_example.py 
```
    * MoveIt!の利用
```
$ roslaunch crane_x7_moveit_config demo.launch 
```

---

### 特定の関節角を入力

```
    js = arm.get_current_joint_values()
    js[0] += 1.0
    arm.set_joint_value_target(js)
    arm.go()                                                    # 実行
    print("hoge")
```
