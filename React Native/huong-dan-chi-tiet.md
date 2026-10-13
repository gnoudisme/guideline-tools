# Hướng dẫn React Native chi tiết

> Tài liệu học React Native từ cơ bản đến navigation, API native, publish app — giải thích chi tiết từng khái niệm kèm ví dụ chạy được thật.

## Mục lục

1. [Giới thiệu](#1-giới-thiệu)
2. [Cài đặt môi trường](#2-cài-đặt-môi-trường)
3. [Tạo project đầu tiên](#3-tạo-project-đầu-tiên)
4. [Tư duy React Native](#4-tư-duy-react-native)
5. [Core Components](#5-core-components)
6. [Styling với StyleSheet](#6-styling-với-stylesheet)
7. [Flexbox trong React Native](#7-flexbox-trong-react-native)
8. [State, Props và Hooks](#8-state-props-và-hooks)
9. [Xử lý sự kiện chạm](#9-xử-lý-sự-kiện-chạm)
10. [Danh sách: FlatList và SectionList](#10-danh-sách-flatlist-và-sectionlist)
11. [TextInput và Form](#11-textinput-và-form)
12. [Navigation](#12-navigation)
13. [Gọi API](#13-gọi-api)
14. [Lưu trữ dữ liệu](#14-lưu-trữ-dữ-liệu)
15. [Quyền và API native](#15-quyền-và-api-native)
16. [Hình ảnh và media](#16-hình-ảnh-và-media)
17. [Animation](#17-animation)
18. [Debug](#18-debug)
19. [Build và publish app](#19-build-và-publish-app)
20. [OTA update với Expo](#20-ota-update-với-expo)
21. [Best practices](#21-best-practices)

---

## 1. Giới thiệu

**React Native** do Facebook phát triển (2015), cho phép viết app mobile **native** (iOS + Android) bằng **JavaScript/TypeScript + React**. Khác với WebView (như Cordova), React Native render ra **component native thật** (UIButton trên iOS, android.widget.Button trên Android).

**Kiến trúc:**
```
JavaScript Thread  ←→  Bridge  ←→  Native Thread (iOS/Android)
   (logic React)      (giao tiếp)    (render UI thật)
```

**Ưu điểm:**
- **Một codebase, hai nền tảng**: viết một lần, chạy cả iOS lẫn Android
- **Hot reload**: sửa code thấy ngay trên máy, không cần build lại
- **Hiệu năng gần native**: UI là native thật, chỉ logic chạy trên JS thread
- **Hệ sinh thái React**: dùng lại tư duy component, hooks từ ReactJS

**Nhược điểm:**
- Tác vụ nặng tính toán trên JS thread có thể giật (giải pháp: Hermes engine, native module, worklet)
- Một số API platform-specific vẫn cần viết native (Java/Kotlin, Swift/ObjC)
- App nặng hơn native thuần một chút

**Khi nào dùng React Native?** App business, social, e-commerce, content — nơi tốc độ phát triển quan trọng hơn tối ưu từng frame. Game 3D nặng, app xử lý video realtime → nên native.

---

## 2. Cài đặt môi trường

### Cách 1: Expo (khuyên dùng cho người mới)

Expo là framework bao quanh React Native, lo hết phần native phức tạp:

```bash
# Cài Expo CLI
npm install -g expo-cli

# Cài app Expo Go trên điện thoại (CH Play / App Store)
# để chạy thử app ngay, không cần Android Studio!
```

### Cách 2: React Native CLI (bare)

Cần cài Android Studio (cho Android) và Xcode (cho iOS, chỉ trên macOS). Phức tạp hơn nhưng toàn quyền kiểm soát native code.

**Tài liệu này dùng Expo** vì cậu dùng Android không có PC — Expo Go cho phép dev trực tiếp trên điện thoại!

---

## 3. Tạo project đầu tiên

```bash
npx create-expo-app my-app
cd my-app
npx expo start
```

Quét mã QR bằng app **Expo Go** trên điện thoại → app chạy ngay!

`App.js`:
```javascript
import { Text, View } from "react-native";

export default function App() {
    return (
        <View style={{ flex: 1, justifyContent: "center", alignItems: "center" }}>
            <Text>Xin chào React Native!</Text>
        </View>
    );
}
```

**Giải thích:**
- `View`: như `<div>` — container cơ bản nhất
- `Text`: mọi chữ PHẢI nằm trong `<Text>` (không render text "trần" như web!)
- `style`: object style, không phải CSS string

---

## 4. Tư duy React Native

Nếu đã biết ReactJS, 80% kiến thức dùng lại được (component, props, state, hooks, JSX). **20% khác biệt:**

| ReactJS (web) | React Native (mobile) |
|---------------|----------------------|
| `<div>`, `<span>` | `<View>` |
| `<p>`, `<h1>` | `<Text>` |
| `<img>` | `<Image>` |
| `<input>` | `<TextInput>` |
| `<button>` | `<Button>` / `<Pressable>` |
| CSS stylesheet | StyleSheet object |
| `onClick` | `onPress` |
| Đơn vị px, %, em | Đơn vị **dp** (density-independent), không có px/em |
| Hover | Không có (mobile không hover!) |

**Không có CSS cascade**: style trong React Native là object JS phẳng, không có selector, không có kế thừa tự động. Mỗi component tự mang style của mình.

---

## 5. Core Components

### View — container

```javascript
import { View } from "react-native";

<View style={{ padding: 16, backgroundColor: "#fff" }}>
    {/* nội dung bên trong */}
</View>
```

### Text — hiển thị chữ

```javascript
import { Text } from "react-native";

<Text style={{ fontSize: 20, fontWeight: "bold", color: "#333" }}>
    Xin chào!
</Text>

// Text lồng nhau (kế thừa style!)
<Text style={{ fontSize: 16 }}>
    Chào <Text style={{ fontWeight: "bold" }}>Dương</Text> nhé!
</Text>

// Cắt dòng
<Text numberOfLines={2} ellipsizeMode="tail">
    Đoạn văn rất dài sẽ bị cắt sau 2 dòng với dấu...
</Text>
```

**Text lồng nhau kế thừa style của cha** — một trong số ít chỗ có "kế thừa" trong React Native.

### Image — hiển thị ảnh

```javascript
import { Image } from "react-native";

// Ảnh từ mạng (BẮT BUỘC chỉ định width/height!)
<Image
    source={{ uri: "https://example.com/anh.jpg" }}
    style={{ width: 200, height: 200, borderRadius: 100 }}
/>

// Ảnh local
<Image source={require("./assets/avatar.png")} style={{ width: 100, height: 100 }} />

// Các chế độ hiển thị
<Image source={...} style={{ width: 200, height: 200 }} resizeMode="cover" />
// cover: lấp đầy, cắt bớt | contain: vừa khít, có khoảng trống
// stretch: kéo giãn | center: giữa nguyên kích thước
```

### ScrollView — cuộn

```javascript
import { ScrollView } from "react-native";

<ScrollView>
    {/* nội dung dài hơn màn hình */}
</ScrollView>

// Scroll ngang
<ScrollView horizontal showsHorizontalScrollIndicator={false}>
    {/* ... */}
</ScrollView>
```

**Lưu ý:** `ScrollView` render **tất cả** children cùng lúc — chỉ dùng cho danh sách ngắn. Danh sách dài → dùng `FlatList` (mục 10).

### Các component khác

```javascript
import { Button, Switch, ActivityIndicator, StatusBar, SafeAreaView } from "react-native";

<Button title="Bấm tôi" onPress={() => alert("Đã bấm!")} color="#007AFF" />
<Switch value={bat} onValueChange={setBat} />  // công tắc
<ActivityIndicator size="large" color="#0000ff" />  // vòng xoay loading
<StatusBar barStyle="dark-content" />  // thanh trạng thái
<SafeAreaView style={{ flex: 1 }}>  // tránh tai thỏ/camera
    {/* ... */}
</SafeAreaView>
```

---

## 6. Styling với StyleSheet

```javascript
import { StyleSheet, Text, View } from "react-native";

function App() {
    return (
        <View style={styles.container}>
            <Text style={styles.tieuDe}>Xin chào</Text>
            <Text style={[styles.chu, styles.chuDo]}>Kết hợp nhiều style</Text>
        </View>
    );
}

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: "#fff",
        padding: 16,
    },
    tieuDe: {
        fontSize: 24,
        fontWeight: "bold",
        marginBottom: 8,
    },
    chu: { fontSize: 16 },
    chuDo: { color: "red" },
});
```

**Tại sao dùng `StyleSheet.create` thay vì object thường?**
1. **Validate**: báo lỗi ngay nếu viết sai tên thuộc tính
2. **Hiệu năng**: style được đăng ký một lần bằng ID, không tạo object mới mỗi render
3. **Tổ chức**: gom style một chỗ, code render gọn

**Kết hợp style:** `style={[styles.a, styles.b]}` — style sau ghi đè style trước (như CSS cascade thu nhỏ). Dùng cho conditional style:
```javascript
<Text style={[styles.chu, isLoi && styles.chuDo]}>...</Text>
```

**Những thuộc tính CSS KHÔNG có trong React Native:** `float`, `position: fixed`, `display: grid`, `cursor`, `z-index` phức tạp, pseudo-selector (`:hover`), media query. Thay vào đó dùng flexbox + Dimensions API.

---

## 7. Flexbox trong React Native

Flexbox là **cách duy nhất** để layout trong React Native — và may mắn là nó mặc định đã bật!

**Khác web:** trong React Native, mọi `View` mặc định là `flexDirection: "column"` (xếp dọc), còn web mặc định là `row`.

```javascript
const styles = StyleSheet.create({
    hang: {
        flexDirection: "row",     // xếp ngang: row | column (mặc định)
        justifyContent: "space-between",  // căn theo trục CHÍNH
        // flex-start | center | flex-end | space-between | space-around | space-evenly
        alignItems: "center",     // căn theo trục PHỤ
        // stretch (mặc định) | flex-start | center | flex-end
    },
    o: {
        flex: 1,  // chiếm phần không gian còn lại (chia đều nếu nhiều đứa cùng flex: 1)
    },
});
```

**Ví dụ layout phổ biến — header:**
```javascript
<View style={{ flexDirection: "row", alignItems: "center", padding: 16 }}>
    <Image source={...} style={{ width: 40, height: 40, borderRadius: 20 }} />
    <View style={{ flex: 1, marginLeft: 12 }}>
        <Text style={{ fontWeight: "bold" }}>Dương</Text>
        <Text style={{ color: "#666" }}>Đang hoạt động</Text>
    </View>
    <Button title="Nhắn" onPress={() => {}} />
</View>
```

**Lấy kích thước màn hình:**
```javascript
import { Dimensions, useWindowDimensions } from "react-native";

const { width, height } = Dimensions.get("window");  // tĩnh

function Component() {
    const { width } = useWindowDimensions();  // tự cập nhật khi xoay màn hình
    return <View style={{ width: width / 2 }} />;
}
```

---

## 8. State, Props và Hooks

Y hệt ReactJS! Mọi hooks (`useState`, `useEffect`, `useContext`...) dùng được.

```javascript
import { useState } from "react";
import { View, Text, Button } from "react-native";

function Dem() {
    const [dem, setDem] = useState(0);
    return (
        <View>
            <Text>Đã bấm {dem} lần</Text>
            <Button title="Bấm" onPress={() => setDem(d => d + 1)} />
        </View>
    );
}
```

---

## 9. Xử lý sự kiện chạm

Mobile không có click/hover — có các gesture chạm:

```javascript
import { Pressable, Text, TouchableOpacity } from "react-native";

// Pressable: hiện đại nhất, linh hoạt nhất
<Pressable
    onPress={() => console.log("Bấm!")}
    onLongPress={() => console.log("Bấm giữ!")}
    style={({ pressed }) => [
        styles.nut,
        pressed && styles.nutDangBam,  // đổi style khi đang bấm!
    ]}
>
    <Text>Bấm tôi</Text>
</Pressable>

// TouchableOpacity: cũ hơn, tự làm mờ khi bấm
<TouchableOpacity onPress={() => {}} activeOpacity={0.6}>
    <Text>Bấm tôi</Text>
</TouchableOpacity>
```

**Nên dùng `Pressable`** cho code mới — nó thay thế tất cả Touchable* (Opacity, Highlight, WithoutFeedback).

---

## 10. Danh sách: FlatList và SectionList

### FlatList — danh sách dài, hiệu năng cao

```javascript
import { FlatList, Text, View } from "react-native";

const DATA = [
    { id: "1", ten: "Dương" },
    { id: "2", ten: "Lan" },
    // ... hàng nghìn item cũng được
];

function DanhSach() {
    function renderItem({ item }) {  // mỗi item render thế nào
        return (
            <View style={styles.hang}>
                <Text>{item.ten}</Text>
            </View>
        );
    }

    return (
        <FlatList
            data={DATA}
            renderItem={renderItem}
            keyExtractor={item => item.id}  // key ổn định!
            onEndReached={() => taiThem()}  // cuộn tới cuối → tải thêm (pagination)
            onEndReachedThreshold={0.5}
            ListEmptyComponent={<Text>Chưa có dữ liệu</Text>}
            ListHeaderComponent={<Text>Danh sách bạn bè</Text>}
            refreshing={dangTaiLai}
            onRefresh={() => taiLai()}  // kéo xuống để refresh!
            ItemSeparatorComponent={() => <View style={{ height: 1, backgroundColor: "#eee" }} />}
        />
    );
}
```

**Tại sao FlatList nhanh?** Nó chỉ render item **đang hiển thị trên màn hình** (+ một ít buffer), item ngoài màn hình được tái chế (recycling). `ScrollView` render tất cả → hàng nghìn item là treo máy.

### SectionList — danh sách có nhóm

```javascript
import { SectionList, Text } from "react-native";

const DATA = [
    { title: "A", data: ["An", "Anh"] },
    { title: "B", data: ["Bình", "Bảo"] },
];

<SectionList
    sections={DATA}
    keyExtractor={(item, index) => item + index}
    renderItem={({ item }) => <Text>{item}</Text>}
    renderSectionHeader={({ section: { title } }) => (
        <Text style={{ fontWeight: "bold" }}>{title}</Text>
    )}
/>
```

---

## 11. TextInput và Form

```javascript
import { useState } from "react";
import { TextInput, View, Button } from "react-native";

function FormDangNhap() {
    const [email, setEmail] = useState("");
    const [matKhau, setMatKhau] = useState("");

    return (
        <View style={{ padding: 16 }}>
            <TextInput
                style={styles.input}
                placeholder="Email"
                value={email}
                onChangeText={setEmail}  // NHẬN TRỰC TIẾP string (không phải event!)
                keyboardType="email-address"  // bàn phím email (@, .com)
                autoCapitalize="none"    // không tự viết hoa
                autoCorrect={false}
            />
            <TextInput
                style={styles.input}
                placeholder="Mật khẩu"
                value={matKhau}
                onChangeText={setMatKhau}
                secureTextEntry  // ẩn thành •••
            />
            <Button title="Đăng nhập" onPress={() => dangNhap(email, matKhau)} />
        </View>
    );
}

const styles = StyleSheet.create({
    input: {
        borderWidth: 1,
        borderColor: "#ccc",
        borderRadius: 8,
        padding: 12,
        marginBottom: 12,
        fontSize: 16,
    },
});
```

**Khác web:** `onChangeText` nhận trực tiếp chuỗi (không phải event object). `keyboardType` đổi loại bàn phím: `"numeric"`, `"phone-pad"`, `"email-address"`, `"default"`.

**Bẫy bàn phím che input:** dùng `KeyboardAvoidingView`:
```javascript
import { KeyboardAvoidingView, Platform } from "react-native";

<KeyboardAvoidingView
    behavior={Platform.OS === "ios" ? "padding" : "height"}
    style={{ flex: 1 }}
>
    {/* form ở đây, bàn phím hiện lên sẽ đẩy form lên */}
</KeyboardAvoidingView>
```

---

## 12. Navigation

App mobile cần chuyển màn hình — dùng **React Navigation**:

```bash
npx expo install @react-navigation/native @react-navigation/native-stack
```

### Stack Navigator — chuyển màn hình chồng lên nhau

```javascript
import { NavigationContainer } from "@react-navigation/native";
import { createNativeStackNavigator } from "@react-navigation/native-stack";
import { Button, Text, View } from "react-native";

const Stack = createNativeStackNavigator();

function TrangChu({ navigation }) {
    return (
        <View style={{ flex: 1, justifyContent: "center", alignItems: "center" }}>
            <Text>Trang chủ</Text>
            <Button
                title="Sang chi tiết"
                onPress={() => navigation.navigate("ChiTiet", { id: 42 })}
            />
        </View>
    );
}

function ChiTiet({ route, navigation }) {
    const { id } = route.params;  // nhận tham số
    return (
        <View style={{ flex: 1, justifyContent: "center", alignItems: "center" }}>
            <Text>Chi tiết #{id}</Text>
            <Button title="Quay lại" onPress={() => navigation.goBack()} />
        </View>
    );
}

export default function App() {
    return (
        <NavigationContainer>
            <Stack.Navigator initialRouteName="TrangChu">
                <Stack.Screen name="TrangChu" component={TrangChu} options={{ title: "Trang chủ" }} />
                <Stack.Screen name="ChiTiet" component={ChiTiet} options={{ title: "Chi tiết" }} />
            </Stack.Navigator>
        </NavigationContainer>
    );
}
```

### Các loại Navigator khác

```bash
npx expo install @react-navigation/bottom-tabs @react-navigation/drawer
```

```javascript
// Bottom Tabs — thanh tab dưới cùng (như Facebook, Zalo)
import { createBottomTabNavigator } from "@react-navigation/bottom-tabs";
const Tab = createBottomTabNavigator();
<Tab.Navigator>
    <Tab.Screen name="Home" component={TrangChu} />
    <Tab.Screen name="CaiDat" component={CaiDat} />
</Tab.Navigator>

// Drawer — menu trượt từ cạnh (như Gmail)
import { createDrawerNavigator } from "@react-navigation/drawer";
```

**Kết hợp:** Tab bên ngoài, mỗi tab chứa một Stack bên trong — pattern phổ biến nhất của app thật.

---

## 13. Gọi API

Y hệt web — dùng `fetch`:

```javascript
import { useState, useEffect } from "react";
import { View, Text, ActivityIndicator, FlatList } from "react-native";

function DanhSachBaiViet() {
    const [data, setData] = useState([]);
    const [loading, setLoading] = useState(true);

    useEffect(() => {
        fetch("https://jsonplaceholder.typicode.com/posts")
            .then(r => r.json())
            .then(setData)
            .catch(console.error)
            .finally(() => setLoading(false));
    }, []);

    if (loading) return <ActivityIndicator size="large" style={{ flex: 1 }} />;

    return (
        <FlatList
            data={data}
            keyExtractor={item => String(item.id)}
            renderItem={({ item }) => (
                <View style={{ padding: 16, borderBottomWidth: 1, borderColor: "#eee" }}>
                    <Text style={{ fontWeight: "bold" }}>{item.title}</Text>
                    <Text>{item.body}</Text>
                </View>
            )}
        />
    );
}
```

---

## 14. Lưu trữ dữ liệu

### AsyncStorage — key-value đơn giản

```bash
npx expo install @react-native-async-storage/async-storage
```

```javascript
import AsyncStorage from "@react-native-async-storage/async-storage";

// Lưu
await AsyncStorage.setItem("token", "abc123");
await AsyncStorage.setItem("user", JSON.stringify({ ten: "Dương" }));

// Đọc
const token = await AsyncStorage.getItem("token");
const user = JSON.parse(await AsyncStorage.getItem("user"));

// Xóa
await AsyncStorage.removeItem("token");
```

**Lưu ý:** AsyncStorage chỉ lưu **string**, chậm với dữ liệu lớn, không mã hóa. Dùng cho: token, setting, cache nhỏ.

### Expo SecureStore — lưu trữ bảo mật

```bash
npx expo install expo-secure-store
```

```javascript
import * as SecureStore from "expo-secure-store";

// Dùng Keychain (iOS) / Keystore (Android) — mã hóa phần cứng!
await SecureStore.setItemAsync("matKhau", "secret123");
const mk = await SecureStore.getItemAsync("matKhau");
```

Dùng cho: mật khẩu, token nhạy cảm, khóa mã hóa.

### SQLite — database local

```bash
npx expo install expo-sqlite
```

Cho dữ liệu có cấu trúc, cần truy vấn phức tạp (như app ghi chú offline, cache lớn).

---

## 15. Quyền và API native

### Xin quyền (permission)

```javascript
import { Camera } from "expo-camera";

const [quyen, xinQuyen] = Camera.useCameraPermissions();

if (!quyen) return <Text>Đang kiểm tra quyền...</Text>;
if (!quyen.granted) {
    return <Button title="Cho phép dùng camera" onPress={xinQuyen} />;
}
// Có quyền → dùng camera
```

**Quy tắc:** luôn giải thích tại sao cần quyền TRƯỚC khi xin. User từ chối → hướng dẫn mở cài đặt.

### Các Expo SDK phổ biến

```bash
npx expo install expo-camera expo-location expo-notifications expo-image-picker
```

```javascript
// Chụp/chọn ảnh
import * as ImagePicker from "expo-image-picker";
const kq = await ImagePicker.launchImageLibraryAsync({ mediaTypes: "images" });
if (!kq.canceled) console.log(kq.assets[0].uri);

// Vị trí
import * as Location from "expo-location";
const { status } = await Location.requestForegroundPermissionsAsync();
const pos = await Location.getCurrentPositionAsync({});
console.log(pos.coords.latitude, pos.coords.longitude);

// Thông báo đẩy
import * as Notifications from "expo-notifications";
await Notifications.scheduleNotificationAsync({
    content: { title: "Nhắc nhở", body: "Uống nước!" },
    trigger: { seconds: 60 },
});
```

---

## 16. Hình ảnh và media

Ngoài `<Image>` cơ bản:

```javascript
// Ảnh với placeholder khi đang tải
import { Image } from "expo-image";  // mạnh hơn Image gốc: cache, blurhash
<Image
    source="https://..."
    placeholder={{ blurhash: "LKO2:N%2Tw=w]~RBVZRi};RPxuwH" }}
    style={{ width: 200, height: 200 }}
/>

// Toàn màn hình xem ảnh: dùng Modal + Image
import { Modal } from "react-native";
```

---

## 17. Animation

### Animated API (có sẵn)

```javascript
import { useRef } from "react";
import { Animated, Button } from "react-native";

function NutPhongTo() {
    const scale = useRef(new Animated.Value(1)).current;

    function phongTo() {
        Animated.spring(scale, { toValue: 1.5, useNativeDriver: true }).start();
    }

    return (
        <Animated.View style={{ transform: [{ scale }] }}>
            <Button title="Phóng to" onPress={phongTo} />
        </Animated.View>
    );
}
```

**`useNativeDriver: true`** — chạy animation trên native thread, không qua bridge → mượt 60fps. Luôn bật khi có thể (chỉ hỗ trợ transform và opacity).

### Reanimated (mạnh hơn)

Thư viện animation hiệu năng cao nhất hiện nay — animation chạy hoàn toàn trên UI thread bằng worklet. Dùng cho gesture phức tạp, animation theo tay người dùng.

---

## 18. Debug

1. **Lắc điện thoại** (hoặc `m` trong terminal Expo) → mở developer menu
2. **Fast Refresh**: sửa code → tự reload, giữ nguyên state!
3. **Console log**: xem trong terminal chạy `npx expo start`
4. **React DevTools**: debug component tree
5. **Lỗi đỏ (RedBox)**: lỗi JS — đọc stack trace, sửa, tự reload

**Mẹo:** khi gặp lỗi lạ, thử `npx expo start -c` (xóa cache).

---

## 19. Build và publish app

### Với Expo (dễ nhất)

```bash
# Build file cài đặt
eas build --platform android  # ra file .aab (lên CH Play)
eas build --platform ios      # cần tài khoản Apple Developer ($99/năm)

# Cần cài EAS CLI trước
npm install -g eas-cli
eas login
eas build:configure
```

### Lên CH Play

1. Build ra file `.aab` bằng EAS
2. Tạo tài khoản Google Play Developer ($25 một lần)
3. Upload `.aab` lên Play Console → điền thông tin → phát hành

### Không dùng Expo (bare workflow)

```bash
cd android && ./gradlew assembleRelease  # ra APK
```
Cần cấu hình signing key — xem docs React Native.

---

## 20. OTA update với Expo

**Điểm mạnh nhất của Expo:** cập nhật JS mà không cần qua review CH Play!

```bash
eas update --branch production --message "Sửa lỗi đăng nhập"
```

User mở app → tự tải bản JS mới → áp dụng ngay. Chỉ code JS/TS được update OTA — đổi native code vẫn phải build lại.

---

## 21. Best practices

1. **Dùng TypeScript** cho project mới — bắt lỗi sớm, IDE gợi ý tốt
2. **Dùng FlatList** cho danh sách dài, không dùng ScrollView + map
3. **Tối ưu render**: `React.memo`, `useMemo`, `useCallback` cho list item
4. **Xử lý đủ trạng thái**: loading, error, empty, success
5. **Không hardcode kích thước** — dùng flexbox + `useWindowDimensions`
6. **Test trên máy thật**, không chỉ giả lập (hiệu năng, gesture khác nhiều)
7. **Xin quyền đúng lúc** — khi user cần tính năng, không phải lúc mở app
8. **Dùng `useNativeDriver: true`** cho animation
9. **Tách logic API vào custom hook** (`useUsers()`), component chỉ render
10. **Dùng EAS Update** cho fix gấp, không đợi review store

---

*React Native = React + tư duy mobile. Nắm chắc flexbox, FlatList, navigation và permission là đủ xây app thật dùng được.*
