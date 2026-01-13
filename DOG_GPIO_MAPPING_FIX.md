# Dog机器人GPIO映射统一修正

## 修改日期
2026-01-08

## 问题
原来的GPIO映射定义与实际需求不一致，导致动作混乱。

## 正确的GPIO映射（已修正）

| 腿的位置 | GPIO | 舵机索引 |
|----------|------|----------|
| 左前腿 | IO39 | [0] LEFT_FRONT_LEG |
| 左后腿 | IO38 | [1] LEFT_REAR_LEG |
| 右前腿 | IO17 | [2] RIGHT_FRONT_LEG |
| 右后腿 | IO18 | [3] RIGHT_REAR_LEG |

## 修改的文件

### 1. `main/boards/dog/config.h`
```cpp
// 修改前
#define LEFT_REAR_LEG_PIN GPIO_NUM_39   // 左后腿
#define LEFT_FRONT_LEG_PIN GPIO_NUM_38  // 左前腿

// 修改后
#define LEFT_FRONT_LEG_PIN GPIO_NUM_39  // 左前腿
#define LEFT_REAR_LEG_PIN GPIO_NUM_38   // 左后腿
```

### 2. `main/boards/dog/dog_movements.h`
```cpp
// 修改前
#define LEFT_REAR_LEG 0   // 左后腿
#define LEFT_FRONT_LEG 1  // 左前腿

// 修改后
#define LEFT_FRONT_LEG 0  // 左前腿 - GPIO 39
#define LEFT_REAR_LEG 1   // 左后腿 - GPIO 38
```

### 3. `main/boards/dog/dog_movements.cc`

#### Init函数参数顺序
```cpp
// 修改前
void Dog::Init(int left_rear_leg, int left_front_leg, int right_front_leg, int right_rear_leg)

// 修改后
void Dog::Init(int left_front_leg, int left_rear_leg, int right_front_leg, int right_rear_leg)
```

#### SetTrims函数参数顺序
```cpp
// 修改前
void Dog::SetTrims(int left_rear_leg, int left_front_leg, int right_front_leg, int right_rear_leg)

// 修改后
void Dog::SetTrims(int left_front_leg, int left_rear_leg, int right_front_leg, int right_rear_leg)
```

#### SayHello函数中的舵机数组
```cpp
// 修改前
int target[SERVO_COUNT] = {
  left_sit,    // [0] LEFT_REAR_LEG
  neutral,     // [1] LEFT_FRONT_LEG
  neutral,     // [2] RIGHT_FRONT_LEG
  right_sit    // [3] RIGHT_REAR_LEG
};

// 修改后
int target[SERVO_COUNT] = {
  neutral,     // [0] LEFT_FRONT_LEG
  left_sit,    // [1] LEFT_REAR_LEG
  neutral,     // [2] RIGHT_FRONT_LEG
  right_sit    // [3] RIGHT_REAR_LEG
};
```

### 4. `main/boards/dog/dog_controller.cc`
```cpp
// 修改前
dog_.Init(LEFT_REAR_LEG_PIN, LEFT_FRONT_LEG_PIN, RIGHT_FRONT_LEG_PIN, RIGHT_REAR_LEG_PIN);

// 修改后
dog_.Init(LEFT_FRONT_LEG_PIN, LEFT_REAR_LEG_PIN, RIGHT_FRONT_LEG_PIN, RIGHT_REAR_LEG_PIN);
```

## 招手动作流程（修正后）

### 舵机数组索引
```
[0] = LEFT_FRONT_LEG  (左前腿 - GPIO 39) ← 招手的腿
[1] = LEFT_REAR_LEG   (左后腿 - GPIO 38)
[2] = RIGHT_FRONT_LEG (右前腿 - GPIO 17)
[3] = RIGHT_REAR_LEG  (右后腿 - GPIO 18)
```

### 动作步骤

| 步骤 | [0]左前腿 | [1]左后腿 | [2]右前腿 | [3]右后腿 |
|------|-----------|-----------|-----------|-----------|
| 第1步：坐下 | 中立 | 向前转(坐下) | 中立 | 向前转(坐下) |
| 第2步：招手 | **来回摆动** | 保持坐下 | 中立 | 保持坐下 |
| 第3步：收手 | 回中立 | 保持坐下 | 中立 | 保持坐下 |
| 第4步：站起 | 中立 | 回中立 | 中立 | 回中立 |

## 测试步骤

```bash
# 1. 编译
./compile_dog.sh

# 2. 烧录
./完整烧录.sh

# 3. 测试走路
self.dog.walk_forward()

# 4. 测试招手
self.dog.say_hello()
```

## 预期效果

✅ 走路动作正常
✅ 坐下时，左后腿和右后腿向前转
✅ 招手时，左前腿来回摆动
✅ 右前腿保持中立不动
✅ 所有动作逻辑正确

## 关键改进

1. **GPIO映射统一**：左前=39, 左后=38, 右前=17, 右后=18
2. **索引定义统一**：数组索引与实际腿的位置对应
3. **函数参数统一**：Init和SetTrims的参数顺序与索引一致
4. **动作逻辑修正**：SayHello中的舵机数组与新索引对应

## 所有动作都已修改 ✅

已修改的动作函数：
- ✅ `WalkForward` - 前进（8个步骤的target数组）
- ✅ `WalkBackward` - 后退（8个步骤的target数组）
- ✅ `TurnRight` - 右转（4个步骤的target数组）
- ✅ `TurnLeft` - 左转（4个步骤的target数组）
- ✅ `SayHello` - 招手（4个步骤的target数组）

所有target数组中的[0]和[1]位置都已互换，使其符合新的索引定义。

## 动作模式不变

虽然修改了舵机索引顺序，但动作逻辑完全不变：
- 前进依然是对角线步态
- 后退依然是前进的逆序
- 转弯依然是对角线力矩
- 招手依然是坐下+左前腿摆动

**只是把数组中的值换了个位置，实际的腿还是做同样的动作！**

---

**重要提醒：** 修改后请务必测试所有动作，确保映射正确！

```bash
# 测试所有动作
self.dog.walk_forward()   # 前进
self.dog.walk_backward()  # 后退
self.dog.turn_right()     # 右转
self.dog.turn_left()      # 左转
self.dog.say_hello()      # 招手
```

