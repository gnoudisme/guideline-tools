# Hướng Dẫn Toàn Diện Expo (React Native)

> Tài liệu tiếng Việt chi tiết về Expo — làm app mobile iOS/Android bằng JavaScript/TypeScript, không cần biết Swift/Kotlin.
> Cập nhật: 2026-10-10 | Tác giả: Capybara 🦫

---

## Mục Lục

1. [Expo Là Gì?](#1-expo-là-gì)
2. [Cài Đặt Môi Trường](#2-cài-đặt-môi-trường)
3. [Tạo Dự Án Đầu Tiên](#3-tạo-dự-án-đầu-tiên)
4. [Expo Go — Chạy App Trên Điện Thoại](#4-expo-go--chạy-app-trên-điện-thoại)
5. [Cấu Trúc Thư Mục Dự Án](#5-cấu-trúc-thư-mục-dự-án)
6. [Components Cơ Bản](#6-components-cơ-bản)
7. [Styling Với StyleSheet](#7-styling-với-stylesheet)
8. [Navigation — Điều Hướng Giữa Các Màn Hình](#8-navigation--điều-hướng-giữa-các-màn-hình)
9. [Quản Lý State](#9-quản-lý-state)
10. [Expo SDK — Thư Viện Có Sẵn](#10-expo-sdk--thư-viện-có-sẵn)
11. [Làm Việc Với API & Dữ Liệu](#11-làm-việc-với-api--dữ-liệu)
12. [Cấu Hình App (app.json)](#12-cấu-hình-app-appjson)
13. [Build APK/IPA Với EAS Build](#13-build-apkpia-với-eas-build)
14. [Cập Nhật OTA Với EAS Update](#14-cập-nhật-ota-với-eas-update)
15. [Đăng App Lên Store](#15-đăng-app-lên-store)
16. [Debug & Xử Lý Lỗi](#16-debug--xử-lý-lỗi)
17. [Best Practices](#17-best-practices)
18. [Tài Liệu Tham Khảo](#18-tài-liệu-tham-khảo)

---

## 1. Expo Là Gì?

### 1.1 Định Nghĩa

**Expo** là framework giúp bạn làm **app mobile thật** (chạy trên iOS và Android) chỉ bằng **JavaScript/TypeScript** — không cần học Swift (iOS) hay Kotlin (Android).

Expo xây dựng trên **React Native** (của Meta), nhưng đơn giản hóa rất nhiều:
- Không cần cài Android Studio / Xcode để bắt đầu
- Chạy thử ngay trên điện thoại thật bằng app **Expo Go**
- Build file APK/IPA trên cloud, không cần máy Mac

### 1.2 Expo vs React Native Thuần

| Tiêu chí | Expo | React Native thuần (CLI) |
|----------|------|--------------------------|
| Cài đặt ban đầu | 5 phút | 30-60 phút (cài SDK...) |
| Chạy thử | Quét QR bằng Expo Go | Cần emulator hoặc máy thật + cable |
| Native modules | Dùng Expo SDK có sẵn | Tự viết native code |
| Build app | EAS Build trên cloud | Cần máy Mac cho iOS |
| Tùy biến sâu | Hạn chế hơn | Toàn quyền |
| Phù hợp | 90% app thông thường | App cần native code đặc biệt |

> 💡 **Khuyên dùng Expo** trừ khi bạn chắc chắn cần viết native code tùy chỉnh sâu.

### 1.3 Expo Managed vs Bare Workflow

| | Managed (khuyên dùng) | Bare |
|---|---|---|
| Code native | Không chạm tới | Có thư mục `android/` `ios/` |
| Thêm thư viện native | Qua Expo SDK hoặc config plugin | Tự link tay |
| Build | EAS Build | Tự build local |

Hầu hết tutorial này dùng **Managed workflow**.

---

## 2. Cài Đặt Môi Trường

### 2.1 Yêu Cầu

- **Node.js** LTS (v18 trở lên) — https://nodejs.org
- Điện thoại iOS hoặc Android (để test)
- Tài khoản Expo (miễn phí) — https://expo.dev/signup

### 2.2 Kiểm Tra Node.js

```bash
node --version   # v18.x.x trở lên
npm --version    # 9.x.x trở lên
```

Nếu chưa có Node.js:
- **Windows/macOS:** Tải từ https://nodejs.org (chọn LTS)
- **Linux:**
```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs
```

### 2.3 Cài Expo CLI (Tùy Chọn)

Expo CLI mới đã tích hợp vào dự án (dùng `npx expo`), không bắt buộc cài global. Nhưng cài global cho tiện:

```bash
npm install -g expo-cli
# hoặc dùng npx mỗi lần (không cần cài):
npx create-expo-app --version
```

### 2.4 Cài App Expo Go Trên Điện Thoại

| Platform | Cách cài |
|----------|----------|
| **Android** | CH Play → tìm **"Expo Go"** |
| **iOS** | App Store → tìm **"Expo Go"** |

> 📱 Expo Go là app "trình chạy thử" — quét QR là chạy được app bạn đang code, không cần build!

---

## 3. Tạo Dự Án Đầu Tiên

### 3.1 Tạo Project

```bash
# Tạo app mới (template mặc định)
npx create-expo-app MyApp

# Với TypeScript (khuyên dùng)
npx create-expo-app MyApp --template blank-typescript

# Với Expo Router (navigation hiện đại)
npx create-expo-app MyApp --template tabs
```

### 3.2 Các Template Có Sẵn

| Template | Mô tả |
|----------|-------|
| `blank` | Trắng tinh, 1 màn hình |
| `blank-typescript` | Trắng + TypeScript |
| `tabs` | Có sẵn bottom tab navigation |
| `bare-minimum` | Bare workflow |

### 3.3 Chạy Dự Án

```bash
cd MyApp
npx expo start
```

Terminal hiện:
```
› Metro waiting on exp://192.168.1.5:8081
› Scan the QR code with Expo Go

› Press a │ open Android
› Press i │ open iOS simulator
› Press w │ open web
```

**Quét mã QR** bằng app Expo Go trên điện thoại → app chạy ngay!

### 3.4 Các Phím Tắt Khi Chạy

| Phím | Chức năng |
|------|-----------|
| `a` | Mở trên Android emulator |
| `i` | Mở trên iOS simulator (chỉ macOS) |
| `w` | Mở trên web browser |
| `r` | Reload app |
| `m` | Mở developer menu |
| `j` | Mở debugger |
| `Shift+m` | Mở menu trên thiết bị |
| `Ctrl+C` | Dừng server |

### 3.5 Sửa Code Đầu Tiên

Mở `App.js` (hoặc `App.tsx`):

```jsx
import { StatusBar } from 'expo-status-bar';
import { StyleSheet, Text, View } from 'react-native';

export default function App() {
  return (
    <View style={styles.container}>
      <Text>Xin chào, đây là app đầu tiên của tôi! 🎉</Text>
      <StatusBar style="auto" />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    alignItems: 'center',
    justifyContent: 'center',
  },
});
```

**Lưu file** → app trên điện thoại **tự động reload**! ⚡

---

## 4. Expo Go — Chạy App Trên Điện Thoại

### 4.1 Cách Kết Nối

**Điều kiện:** Điện thoại và máy tính **cùng mạng WiFi**.

1. Chạy `npx expo start`
2. Mở app **Expo Go** trên điện thoại
3. **Android:** Bấm "Scan QR code" → quét mã trên terminal
4. **iOS:** Mở app Camera → quét mã QR → bấm link hiện ra

### 4.2 Nếu Không Cùng WiFi

Dùng chế độ **tunnel** (chạy qua internet):

```bash
npx expo start --tunnel
```

> ⚠️ Tunnel chậm hơn LAN, chỉ dùng khi cần.

### 4.3 Developer Menu

**Lắc điện thoại** (hoặc bấm `m` trong terminal) để mở menu:
- **Reload:** Tải lại app
- **Debug JS Remotely:** Debug bằng Chrome DevTools
- **Toggle Performance Monitor:** Xem FPS, RAM

### 4.4 Giới Hạn Của Expo Go

Expo Go **không chạy được** nếu app dùng:
- Native module tùy chỉnh (không có trong Expo SDK)
- Cấu hình native đặc biệt

→ Khi đó cần build **development build** (xem phần 13).

---

## 5. Cấu Trúc Thư Mục Dự Án

### 5.1 Template Blank

```
MyApp/
├── App.js                  # File chính (entry point)
├── app.json                # Cấu hình Expo
├── package.json            # Dependencies
├── babel.config.js         # Cấu hình Babel
├── assets/                 # Ảnh, icon, splash
│   ├── icon.png
│   ├── splash.png
│   └── ...
└── node_modules/
```

### 5.2 Template Tabs (Expo Router)

```
MyApp/
├── app/                    # 📁 Mỗi file = 1 màn hình!
│   ├── _layout.tsx         # Layout chung (tab bar)
│   ├── index.tsx           # Màn hình home (route: /)
│   ├── explore.tsx         # Màn hình explore (route: /explore)
│   └── (tabs)/
│       ├── _layout.tsx
│       └── ...
├── components/             # Components dùng chung
├── constants/              # Màu sắc, config
├── hooks/                  # Custom hooks
├── assets/
├── app.json
└── package.json
```

### 5.3 Quy Ước Đặt Tên

- **Components:** PascalCase — `UserCard.js`, `ProductList.tsx`
- **Screens:** PascalCase — `HomeScreen.js`
- **Utils/helpers:** camelCase — `formatDate.js`, `api.js`
- **Constants:** UPPER_SNAKE — hoặc file `constants/Colors.ts`

---

## 6. Components Cơ Bản

React Native **không dùng** thẻ HTML (`div`, `p`...). Thay vào đó:

### 6.1 Bảng Đối Chiếu HTML → React Native

| HTML | React Native | Dùng để |
|------|--------------|---------|
| `<div>` | `<View>` | Khung chứa |
| `<p>`, `<span>`, `<h1>` | `<Text>` | Văn bản (MỌI chữ phải trong `<Text>`) |
| `<img>` | `<Image>` | Ảnh |
| `<input>` | `<TextInput>` | Ô nhập liệu |
| `<button>` | `<Button>` hoặc `<Pressable>` | Nút bấm |
| `<ul>` | `<FlatList>` | Danh sách dài (tối ưu) |
| `<div scroll>` | `<ScrollView>` | Cuộn |

### 6.2 Ví Dụ Tổng Hợp

```jsx
import {
  View, Text, Image, TextInput,
  Button, Pressable, ScrollView, Alert,
  StyleSheet
} from 'react-native';
import { useState } from 'react';

export default function App() {
  const [name, setName] = useState('');

  return (
    <ScrollView style={styles.container}>
      {/* Văn bản */}
      <Text style={styles.title}>Đăng ký tài khoản</Text>

      {/* Ảnh */}
      <Image
        source={{ uri: 'https://picsum.photos/200' }}
        style={styles.avatar}
      />
      {/* Hoặc ảnh local: */}
      {/* <Image source={require('./assets/avatar.png')} /> */}

      {/* Ô nhập */}
      <TextInput
        style={styles.input}
        placeholder="Nhập tên của bạn"
        value={name}
        onChangeText={setName}
      />

      {/* Nút bấm cơ bản */}
      <Button
        title="Đăng ký"
        onPress={() => Alert.alert('Xin chào', `Chào ${name}!`)}
      />

      {/* Nút tùy chỉnh đẹp hơn */}
      <Pressable
        style={styles.myButton}
        onPress={() => console.log('Đã bấm!')}
      >
        <Text style={styles.buttonText}>Nút custom</Text>
      </Pressable>
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 20, backgroundColor: '#fff' },
  title: { fontSize: 24, fontWeight: 'bold', marginBottom: 16 },
  avatar: { width: 100, height: 100, borderRadius: 50, marginBottom: 16 },
  input: {
    borderWidth: 1, borderColor: '#ccc', borderRadius: 8,
    padding: 12, marginBottom: 16, fontSize: 16,
  },
  myButton: {
    backgroundColor: '#007AFF', padding: 14,
    borderRadius: 8, alignItems: 'center', marginTop: 12,
  },
  buttonText: { color: '#fff', fontSize: 16, fontWeight: '600' },
});
```

### 6.3 FlatList — Danh Sách Hiệu Năng Cao

Dùng `FlatList` thay vì `map()` khi danh sách dài (tự động virtualize):

```jsx
import { FlatList, View, Text, StyleSheet } from 'react-native';

const DATA = [
  { id: '1', title: 'Táo' },
  { id: '2', title: 'Chuối' },
  { id: '3', title: 'Cam' },
];

function Item({ title }) {
  return (
    <View style={styles.item}>
      <Text style={styles.itemText}>{title}</Text>
    </View>
  );
}

export default function App() {
  return (
    <FlatList
      data={DATA}
      renderItem={({ item }) => <Item title={item.title} />}
      keyExtractor={item => item.id}
    />
  );
}
```

---

## 7. Styling Với StyleSheet

### 7.1 Khác Biệt Với CSS Web

- Không có `px`, `em` — dùng **số thuần** (đơn vị dp)
- Thuộc tính viết **camelCase**: `backgroundColor` (không phải `background-color`)
- Dùng **Flexbox** làm mặc định (giống CSS nhưng đơn giản hơn)
- Không có kế thừa style phức tạp

### 7.2 Flexbox Cơ Bản

```jsx
import { View, StyleSheet } from 'react-native';

// Xếp ngang (row) vs xếp dọc (column)
<View style={styles.row}>
  <View style={styles.box} />
  <View style={styles.box} />
</View>

const styles = StyleSheet.create({
  row: {
    flexDirection: 'row',        // 'row' | 'column' (mặc định column)
    justifyContent: 'space-between', // căn theo trục chính
    alignItems: 'center',        // căn theo trục phụ
  },
  box: {
    width: 50, height: 50, backgroundColor: 'tomato',
  },
});
```

**Các giá trị `justifyContent`:** `flex-start`, `flex-end`, `center`, `space-between`, `space-around`, `space-evenly`

### 7.3 Style Có Điều Kiện & Gộp Style

```jsx
// Gộp nhiều style (style sau ghi đè style trước)
<View style={[styles.box, styles.red, isActive && styles.active]} />

// Style động
<Text style={{ color: isError ? 'red' : 'black', fontSize: 16 }}>
  Thông báo
</Text>
```

### 7.4 Dimensions — Lấy Kích Thước Màn Hình

```jsx
import { Dimensions } from 'react-native';

const { width, height } = Dimensions.get('window');
console.log(`Màn hình: ${width} x ${height}`);

// Responsive: box rộng 80% màn hình
<View style={{ width: width * 0.8, height: 100 }} />
```

---

## 8. Navigation — Điều Hướng Giữa Các Màn Hình

### 8.1 Expo Router (Khuyên Dùng — Hiện Đại)

Expo Router dùng **file-based routing**: mỗi file trong `app/` = 1 màn hình.

```
app/
├── _layout.tsx       # Layout gốc
├── index.tsx         # → route "/"
├── login.tsx         # → route "/login"
├── profile.tsx       # → route "/profile"
└── settings/
    └── index.tsx     # → route "/settings"
```

**Cài đặt:**
```bash
npx expo install expo-router
```

**`app/_layout.tsx`:**
```tsx
import { Stack } from 'expo-router';

export default function RootLayout() {
  return (
    <Stack>
      <Stack.Screen name="index" options={{ title: 'Trang chủ' }} />
      <Stack.Screen name="login" options={{ title: 'Đăng nhập' }} />
    </Stack>
  );
}
```

**Chuyển màn hình:**
```tsx
// app/index.tsx
import { Link, router } from 'expo-router';
import { View, Text, Pressable } from 'react-native';

export default function Home() {
  return (
    <View>
      {/* Cách 1: Link component */}
      <Link href="/login">
        <Text>Đi tới đăng nhập</Text>
      </Link>

      {/* Cách 2: router.push (trong hàm) */}
      <Pressable onPress={() => router.push('/profile')}>
        <Text>Xem profile</Text>
      </Pressable>

      {/* Quay lại */}
      <Pressable onPress={() => router.back()}>
        <Text>← Quay lại</Text>
      </Pressable>
    </View>
  );
}
```

**Truyền params:**
```tsx
// Đi kèm params
router.push({ pathname: '/profile', params: { userId: '123' } });

// Nhận params ở màn hình đích
import { useLocalSearchParams } from 'expo-router';
const { userId } = useLocalSearchParams();
```

### 8.2 Tab Navigation

```
app/
├── _layout.tsx
└── (tabs)/
    ├── _layout.tsx     # Định nghĩa tab bar
    ├── index.tsx       # Tab 1: Home
    └── settings.tsx    # Tab 2: Settings
```

```tsx
// app/(tabs)/_layout.tsx
import { Tabs } from 'expo-router';
import { Ionicons } from '@expo/vector-icons';

export default function TabLayout() {
  return (
    <Tabs>
      <Tabs.Screen
        name="index"
        options={{
          title: 'Trang chủ',
          tabBarIcon: ({ color }) => (
            <Ionicons name="home" size={24} color={color} />
          ),
        }}
      />
      <Tabs.Screen
        name="settings"
        options={{
          title: 'Cài đặt',
          tabBarIcon: ({ color }) => (
            <Ionicons name="settings" size={24} color={color} />
          ),
        }}
      />
    </Tabs>
  );
}
```

### 8.3 React Navigation (Cách Truyền Thống)

Nếu không dùng Expo Router:

```bash
npx expo install @react-navigation/native @react-navigation/native-stack
npx expo install react-native-screens react-native-safe-area-context
```

```jsx
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';

const Stack = createNativeStackNavigator();

function HomeScreen({ navigation }) {
  return (
    <Button
      title="Đi tới Chi tiết"
      onPress={() => navigation.navigate('Details', { itemId: 42 })}
    />
  );
}

function DetailsScreen({ route }) {
  const { itemId } = route.params;
  return <Text>Item ID: {itemId}</Text>;
}

export default function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Details" component={DetailsScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```

---

## 9. Quản Lý State

### 9.1 useState — State Đơn Giản

```jsx
import { useState } from 'react';

const [count, setCount] = useState(0);
const [user, setUser] = useState({ name: '', age: 0 });

// Cập nhật object (phải spread)
setUser({ ...user, name: 'An' });
```

### 9.2 Context — State Dùng Chung

```jsx
// contexts/AuthContext.js
import { createContext, useContext, useState } from 'react';

const AuthContext = createContext();

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);

  const login = (userData) => setUser(userData);
  const logout = () => setUser(null);

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

export const useAuth = () => useContext(AuthContext);
```

```jsx
// App.js
import { AuthProvider } from './contexts/AuthContext';

export default function App() {
  return (
    <AuthProvider>
      {/* các màn hình */}
    </AuthProvider>
  );
}

// Dùng ở màn hình bất kỳ:
const { user, login } = useAuth();
```

### 9.3 AsyncStorage — Lưu Local

```bash
npx expo install @react-native-async-storage/async-storage
```

```jsx
import AsyncStorage from '@react-native-async-storage/async-storage';

// Lưu
await AsyncStorage.setItem('token', 'abc123');
await AsyncStorage.setItem('user', JSON.stringify({ name: 'An' }));

// Đọc
const token = await AsyncStorage.getItem('token');
const user = JSON.parse(await AsyncStorage.getItem('user'));

// Xóa
await AsyncStorage.removeItem('token');
```

---

## 10. Expo SDK — Thư Viện Có Sẵn

Expo SDK cung cấp sẵn hàng chục module native — **không cần config**:

### 10.1 Bảng Các Module Phổ Biến

| Module | Chức năng | Cài đặt |
|--------|-----------|---------|
| `expo-camera` | Chụp ảnh, quét QR | `npx expo install expo-camera` |
| `expo-image-picker` | Chọn ảnh từ thư viện | `npx expo install expo-image-picker` |
| `expo-location` | GPS, vị trí | `npx expo install expo-location` |
| `expo-notifications` | Push notification | `npx expo install expo-notifications` |
| `expo-av` | Phát audio/video | `npx expo install expo-av` |
| `expo-haptics` | Rung phản hồi | `npx expo install expo-haptics` |
| `expo-sharing` | Chia sẻ file | `npx expo install expo-sharing` |
| `expo-file-system` | Đọc/ghi file | `npx expo install expo-file-system` |
| `expo-secure-store` | Lưu token bảo mật | `npx expo install expo-secure-store` |
| `expo-local-authentication` | Vân tay / Face ID | `npx expo install expo-local-authentication` |

### 10.2 Ví Dụ: Chụp Ảnh

```jsx
import { Camera, CameraType } from 'expo-camera';
import { useState, useRef } from 'react';
import { Button, View, Text } from 'react-native';

export default function CameraScreen() {
  const [permission, requestPermission] = Camera.useCameraPermissions();
  const cameraRef = useRef(null);

  if (!permission?.granted) {
    return (
      <View>
        <Text>Cần quyền camera</Text>
        <Button title="Cấp quyền" onPress={requestPermission} />
      </View>
    );
  }

  const takePhoto = async () => {
    const photo = await cameraRef.current.takePictureAsync();
    console.log('Ảnh:', photo.uri);
  };

  return (
    <View style={{ flex: 1 }}>
      <Camera style={{ flex: 1 }} type={CameraType.back} ref={cameraRef} />
      <Button title="Chụp 📸" onPress={takePhoto} />
    </View>
  );
}
```

### 10.3 Ví Dụ: Lấy Vị Trí GPS

```jsx
import * as Location from 'expo-location';
import { useState, useEffect } from 'react';
import { Text } from 'react-native';

export default function LocationScreen() {
  const [location, setLocation] = useState(null);

  useEffect(() => {
    (async () => {
      const { status } = await Location.requestForegroundPermissionsAsync();
      if (status !== 'granted') return;

      const loc = await Location.getCurrentPositionAsync({});
      setLocation(loc.coords);
    })();
  }, []);

  return (
    <Text>
      {location
        ? `Vĩ độ: ${location.latitude}, Kinh độ: ${location.longitude}`
        : 'Đang lấy vị trí...'}
    </Text>
  );
}
```

### 10.4 Ví Dụ: Lưu Token Bảo Mật

```jsx
import * as SecureStore from 'expo-secure-store';

// SecureStore mã hóa trên thiết bị — an toàn hơn AsyncStorage
await SecureStore.setItemAsync('auth_token', 'secret123');
const token = await SecureStore.getItemAsync('auth_token');
await SecureStore.deleteItemAsync('auth_token');
```

---

## 11. Làm Việc Với API & Dữ Liệu

### 11.1 Fetch API Cơ Bản

```jsx
import { useState, useEffect } from 'react';
import { View, Text, FlatList, ActivityIndicator } from 'react-native';

export default function UserList() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch('https://jsonplaceholder.typicode.com/users')
      .then(res => res.json())
      .then(data => {
        setUsers(data);
        setLoading(false);
      })
      .catch(err => {
        console.error(err);
        setLoading(false);
      });
  }, []);

  if (loading) return <ActivityIndicator size="large" />;

  return (
    <FlatList
      data={users}
      keyExtractor={item => String(item.id)}
      renderItem={({ item }) => (
        <View style={{ padding: 12, borderBottomWidth: 1 }}>
          <Text style={{ fontWeight: 'bold' }}>{item.name}</Text>
          <Text>{item.email}</Text>
        </View>
      )}
    />
  );
}
```

### 11.2 POST Dữ Liệu

```jsx
const login = async (email, password) => {
  try {
    const res = await fetch('https://api.example.com/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email, password }),
    });

    if (!res.ok) throw new Error('Đăng nhập thất bại');

    const data = await res.json();
    await SecureStore.setItemAsync('token', data.token);
    return data;
  } catch (err) {
    Alert.alert('Lỗi', err.message);
  }
};
```

### 11.3 Dùng Axios

```bash
npm install axios
```

```jsx
import axios from 'axios';

// Tạo instance dùng chung
const api = axios.create({
  baseURL: 'https://api.example.com',
  timeout: 10000,
  headers: { 'Content-Type': 'application/json' },
});

// Tự động gắn token
api.interceptors.request.use(async (config) => {
  const token = await SecureStore.getItemAsync('token');
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

// Dùng:
const { data } = await api.get('/users');
await api.post('/login', { email, password });
```

---

## 12. Cấu Hình App (app.json)

### 12.1 File app.json Cơ Bản

```json
{
  "expo": {
    "name": "My Awesome App",
    "slug": "my-awesome-app",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "userInterfaceStyle": "automatic",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    },
    "assetBundlePatterns": ["**/*"],
    "ios": {
      "supportsTablet": true,
      "bundleIdentifier": "com.mycompany.myapp"
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#ffffff"
      },
      "package": "com.mycompany.myapp"
    },
    "web": {
      "favicon": "./assets/favicon.png"
    }
  }
}
```

### 12.2 Các Trường Quan Trọng

| Trường | Ý nghĩa |
|--------|---------|
| `name` | Tên hiển thị trên điện thoại |
| `slug` | Tên URL-friendly (dùng cho Expo) |
| `version` | Phiên bản app (1.0.0, 1.0.1...) |
| `orientation` | `portrait` / `landscape` / `default` |
| `icon` | Icon app (1024x1024 PNG) |
| `splash` | Màn hình chờ khi mở app |
| `ios.bundleIdentifier` | ID duy nhất trên App Store (dạng reverse domain) |
| `android.package` | ID duy nhất trên Play Store |

### 12.3 Xin Quyền (Permissions)

```json
{
  "expo": {
    "ios": {
      "infoPlist": {
        "NSCameraUsageDescription": "App cần camera để chụp ảnh đại diện",
        "NSLocationWhenInUseUsageDescription": "App cần vị trí để tìm quán gần bạn"
      }
    },
    "android": {
      "permissions": ["CAMERA", "ACCESS_FINE_LOCATION"]
    }
  }
}
```

> ⚠️ iOS **bắt buộc** có description tiếng Anh (hoặc ngôn ngữ user) — không có sẽ bị Apple reject!

---

## 13. Build APK/IPA Với EAS Build

### 13.1 EAS Là Gì?

**EAS (Expo Application Services)** là dịch vụ cloud của Expo để:
- **EAS Build:** Build file APK (Android) / IPA (iOS) trên server Expo
- **EAS Update:** Đẩy update JS qua OTA (không cần qua store review)
- **EAS Submit:** Nộp app lên App Store / Play Store tự động

### 13.2 Cài Đặt EAS CLI

```bash
npm install -g eas-cli
eas login   # đăng nhập tài khoản Expo
```

### 13.3 Cấu Hình Build

```bash
cd MyApp
eas build:configure
```

Tạo file `eas.json`:
```json
{
  "cli": { "version": ">= 5.0.0" },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal"
    },
    "preview": {
      "distribution": "internal",
      "android": { "buildType": "apk" }
    },
    "production": {}
  },
  "submit": {
    "production": {}
  }
}
```

### 13.4 Build Các Loại

```bash
# Development build (để test native module, thay Expo Go)
eas build --profile development --platform android

# Preview APK (gửi cho tester, cài trực tiếp)
eas build --profile preview --platform android

# Production (để đăng store)
eas build --profile production --platform all
```

Quá trình build mất **10-30 phút** trên cloud. Xong → tải file APK/IPA về!

### 13.5 Development Build vs Expo Go

| | Expo Go | Development Build |
|---|---|---|
| Cài | Từ CH Play/App Store | Build riêng cho app bạn |
| Native module custom | ❌ Không chạy được | ✅ Chạy được |
| Tốc độ bắt đầu | Nhanh | Cần build 1 lần đầu (~15 phút) |

```bash
# Tạo dev build rồi cài lên máy:
eas build --profile development --platform android
# → Tải APK → cài vào điện thoại
# → Chạy: npx expo start --dev-client
# → Quét QR bằng DEV BUILD (không phải Expo Go)
```

---

## 14. Cập Nhật OTA Với EAS Update

### 14.1 OTA Là Gì?

**Over-the-Air Update:** Đẩy bản cập nhật JavaScript trực tiếp tới user **mà không cần qua App Store review** (mất 2-7 ngày)!

> ⚠️ Chỉ update được phần JS. Nếu đổi native code (thêm thư viện native, đổi app.json) → phải build lại và đăng store.

### 14.2 Cài Đặt

```bash
npx expo install expo-updates
eas update:configure
```

### 14.3 Đẩy Update

```bash
# Đẩy update lên kênh production
eas update --branch production --message "Sửa lỗi đăng nhập"
```

User mở app → tự động tải bản mới → áp dụng khi khởi động lại app!

### 14.4 Các Kênh (Channels/Branches)

```bash
# Tạo kênh staging để test trước
eas branch:create staging

# Đẩy lên staging
eas update --branch staging --message "Test tính năng mới"

# OK rồi mới đẩy production
eas update --branch production --message "Release v1.1"
```

---

## 15. Đăng App Lên Store

### 15.1 Chuẩn Bị

**Android (Google Play):**
1. Tạo tài khoản Google Play Developer ($25 một lần)
2. Tạo app mới trên Play Console
3. Lấy **service account key** (JSON)

**iOS (App Store):**
1. Tài khoản Apple Developer ($99/năm)
2. Tạo App ID, certificates trên App Store Connect

### 15.2 Submit Tự Động Với EAS

```bash
# Android
eas submit --platform android
# → Hỏi file APK/AAB → tự upload lên Play Console

# iOS
eas submit --platform ios
# → Tự upload lên App Store Connect
```

### 15.3 Checklist Trước Khi Đăng

- [ ] Icon đầy đủ kích thước (1024x1024)
- [ ] Splash screen đẹp
- [ ] Tên app, mô tả rõ ràng
- [ ] Screenshot cho từng kích thước màn hình
- [ ] Privacy policy URL (bắt buộc nếu thu thập dữ liệu)
- [ ] Test kỹ trên cả iOS và Android
- [ ] Version number tăng dần
- [ ] Không có `console.log` dư thừa

---

## 16. Debug & Xử Lý Lỗi

### 16.1 Các Lỗi Thường Gặp

| Lỗi | Nguyên nhân | Cách fix |
|-----|-------------|----------|
| `Unable to resolve module` | Thiếu package | `npx expo install ten-package` |
| `Metro bundler error` | Cache cũ | `npx expo start -c` (xóa cache) |
| QR không quét được | Khác mạng WiFi | Dùng `--tunnel` hoặc cùng WiFi |
| `Network request failed` | API http (không https) | Android 9+ chặn http — dùng https hoặc config |
| App trắng màn hình | Lỗi JS khi render | Xem log terminal, kiểm tra console.error |

### 16.2 Xóa Cache

```bash
# Xóa Metro cache
npx expo start -c

# Xóa node_modules cài lại
rm -rf node_modules
npm install

# Xóa cache Expo
npx expo install --fix
```

### 16.3 Xem Log Chi Tiết

```bash
# Log Android (cần adb)
npx expo run:android  # rồi xem logcat

# Log iOS (chỉ macOS)
npx expo run:ios
```

### 16.4 React DevTools

```bash
# Bấm j khi đang chạy expo start
# → Mở debugger trong Chrome
# → Xem component tree, props, state
```

---

## 17. Best Practices

### 17.1 Cấu Trúc Code

```
src/
├── app/              # Screens (expo-router)
├── components/       # UI components dùng chung
│   ├── Button.tsx
│   └── Card.tsx
├── hooks/            # Custom hooks
├── services/         # API calls
│   └── api.ts
├── contexts/         # Global state
├── utils/            # Helpers
└── constants/        # Colors, config
```

### 17.2 TypeScript

Dùng TypeScript từ đầu để ít bug hơn:

```tsx
// Định nghĩa kiểu cho props
interface ButtonProps {
  title: string;
  onPress: () => void;
  disabled?: boolean;
}

export function MyButton({ title, onPress, disabled = false }: ButtonProps) {
  // ...
}
```

### 17.3 Tối Ưu Hiệu Năng

- Dùng `FlatList` thay vì `ScrollView` + `map` cho danh sách dài
- Dùng `React.memo` cho component render nhiều
- Dùng `useMemo` / `useCallback` cho tính toán nặng
- Nén ảnh trước khi đưa vào app (dùng WebP nếu được)
- Không để `console.log` trong production

### 17.4 Bảo Mật

- Lưu token trong `expo-secure-store` (không phải AsyncStorage)
- Không hardcode API key vào code — dùng `app.config.js` + biến môi trường:
```js
// app.config.js
export default {
  expo: {
    // ...
    extra: {
      apiUrl: process.env.API_URL,
    },
  },
};

// Dùng trong code:
import Constants from 'expo-constants';
const apiUrl = Constants.expoConfig.extra.apiUrl;
```

---

## 18. Tài Liệu Tham Khảo

- 🏠 Trang chủ Expo: https://expo.dev
- 📖 Docs: https://docs.expo.dev
- 🧩 Expo SDK reference: https://docs.expo.dev/versions/latest/
- 🛠️ EAS docs: https://docs.expo.dev/eas/
- 🧭 Expo Router: https://docs.expo.dev/router/introduction/
- 💬 Forums: https://forums.expo.dev
- 📺 YouTube: tìm "Expo React Native tutorial"

---

*📝 Viết bởi **Capybara** 🦫 cho **Tiểu Dươngg***
*📅 Cập nhật: 2026-10-10*
