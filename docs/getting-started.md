---
id: environment-setup
title: Get Started with React Native
hide_table_of_contents: true
---

import PlatformSupport from '@site/src/theme/PlatformSupport';
import BoxLink from '@site/src/theme/BoxLink';

**React Native allows developers who know React to create native apps.** At the same time, native developers can use React Native to gain parity between native platforms by writing common features once.

We believe that the best way to experience React Native is through a **Framework**, a toolbox with all the necessary APIs to let you build production ready apps.

You can also use React Native without a Framework, however we’ve found that most developers benefit from using a React Native Framework like [Expo](https://expo.dev). Expo provides features like file-based routing, high-quality universal libraries, and the ability to write plugins that modify native code without having to manage native files.

<details>
<summary>Can I use React Native without a Framework?</summary>

Yes. You can use React Native without a Framework. **However, if you’re building a new app with React Native, we recommend using a Framework.**

In short, you’ll be able to spend time writing your app instead of writing an entire Framework yourself in addition to your app.

The React Native community has spent years refining approaches to navigation, accessing native APIs, dealing with native dependencies, and more. Most apps need these core features. A React Native Framework provides them from the start of your app.

Without a Framework, you’ll either have to write your own solutions to implement core features, or you’ll have to piece together a collection of pre-existing libraries to create a skeleton of a Framework. This takes real work, both when starting your app, then later when maintaining it.

If your app has unusual constraints that are not served well by a Framework, or you prefer to solve these problems yourself, you can make a React Native app without a Framework using Android Studio, Xcode. If you’re interested in this path, learn how to [set up your environment](set-up-your-environment) and how to [get started without a framework](getting-started-without-a-framework).

</details>

## Start a new React Native project with Expo

<PlatformSupport platforms={['android', 'ios', 'tv', 'web']} />

Expo is a production-grade React Native Framework. Expo provides developer tooling that makes developing apps easier, such as file-based routing, a standard library of native modules, and much more.

Expo's Framework is free and open source, with an active community on [GitHub](https://github.com/expo) and [Discord](https://chat.expo.dev). The Expo team works in close collaboration with the React Native team at Meta to bring the latest React Native features to the Expo SDK.

The team at Expo also provides Expo Application Services (EAS), an optional set of services that complements Expo, the Framework, in each step of the development process.

To create a new Expo project, run the following in your terminal:

```shell
npx create-expo-app@latest
```

Once you’ve created your app, check out the rest of Expo’s getting started guide to start developing your app.

<BoxLink href="https://docs.expo.dev/get-started/set-up-your-environment">Continue with Expo</BoxLink>
FOFO/
├─ package.json
├─ App.js
├─ firebase.js
├─ app.json
├─ src/
│  ├─ screens/
│  │  ├─ FeedScreen.js
│  │  ├─ UploadScreen.js
│  │  └─ ProfileScreen.js
│  └─ components/
│     └─ VideoItem.js
{
  "name": "FOFO",
  "version": "1.0.0",
  "main": "node_modules/expo/AppEntry.js",
  "scripts": {
    "start": "expo start",
    "android": "expo run:android",
    "web": "expo start --web"
  },
  "dependencies": {
    "expo": "~48.0.0",
    "expo-av": "~13.0.0",
    "expo-image-picker": "~14.0.0",
    "firebase": "^9.23.0",
    "react": "18.2.0",
    "react-native": "0.71.8",
    "react-native-gesture-handler": "^2.10.0",
    "@react-navigation/native": "^6.1.6",
    "@react-navigation/bottom-tabs": "^6.5.7",
    "@react-navigation/native-stack": "^6.9.12"
  },
  "devDependencies": {
    "@babel/core": "^7.20.0"
  },
  "private": true
}
// firebase.js
import { initializeApp } from "firebase/app";
import { getFirestore } from "firebase/firestore";
import { getStorage } from "firebase/storage";
import { getAuth, signInAnonymously } from "firebase/auth";

const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_AUTH_DOMAIN",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_STORAGE_BUCKET",
  messagingSenderId: "YOUR_MSG_SENDER_ID",
  appId: "YOUR_APP_ID"
};

const app = initializeApp(firebaseConfig);
export const db = getFirestore(app);
export const storage = getStorage(app);
export const auth = getAuth(app);

// simple anonymous sign-in
export const signInAnonymouslyIfNeeded = async () => {
  if (!auth.currentUser) {
    try {
      await signInAnonymously(auth);
    } catch (e) {
      console.warn("Firebase auth error", e);
    }
  }
};
// App.js
import React, { useEffect } from "react";
import { NavigationContainer } from "@react-navigation/native";
import { createBottomTabNavigator } from "@react-navigation/bottom-tabs";
import FeedScreen from "./src/screens/FeedScreen";
import UploadScreen from "./src/screens/UploadScreen";
import ProfileScreen from "./src/screens/ProfileScreen";
import { signInAnonymouslyIfNeeded } from "./firebase";

const Tab = createBottomTabNavigator();

export default function App() {
  useEffect(() => {
    signInAnonymouslyIfNeeded();
  }, []);

  return (
    <NavigationContainer>
      <Tab.Navigator screenOptions={{ headerShown: false }}>
        <Tab.Screen name="Feed" component={FeedScreen} />
        <Tab.Screen name="Upload" component={UploadScreen} />
        <Tab.Screen name="Profile" component={ProfileScreen} />
      </Tab.Navigator>
    </NavigationContainer>
  );
}
// src/components/VideoItem.js
import React, { useRef, useEffect, useState } from "react";
import { View, Text, TouchableOpacity, StyleSheet, Dimensions } from "react-native";
import { Video } from "expo-av";
import { doc, updateDoc, arrayUnion, arrayRemove } from "firebase/firestore";
import { db, auth } from "../../firebase";

const { height: WINDOW_HEIGHT } = Dimensions.get("window");

export default function VideoItem({ item, isActive }) {
  const videoRef = useRef(null);
  const [liked, setLiked] = useState(item.likes?.includes(auth.currentUser?.uid) || false);
  const [likesCount, setLikesCount] = useState(item.likes?.length || 0);

  useEffect(() => {
    const v = videoRef.current;
    if (!v) return;
    if (isActive) {
      v.playAsync().catch(()=>{});
    } else {
      v.pauseAsync().catch(()=>{});
    }
  }, [isActive]);

  const toggleLike = async () => {
    const docRef = doc(db, "videos", item.id);
    try {
      if (liked) {
        await updateDoc(docRef, { likes: arrayRemove(auth.currentUser.uid) });
        setLikesCount(c => c - 1);
      } else {
        await updateDoc(docRef, { likes: arrayUnion(auth.currentUser.uid) });
        setLikesCount(c => c + 1);
      }
      setLiked(!liked);
    } catch (e) {
      console.warn("like error", e);
    }
  };

  return (
    <View style={styles.container}>
      <Video
        ref={videoRef}
        source={{ uri: item.videoURL }}
        style={styles.video}
        resizeMode="cover"
        isLooping
        shouldPlay={false} // controlled by isActive effect
        useNativeControls={false}
        rate={1.0}
      />
      <View style={styles.rightColumn}>
        <TouchableOpacity onPress={toggleLike} style={styles.iconBtn}>
          <Text style={{fontSize:18}}>{liked ? "❤️" : "🤍"}</Text>
          <Text style={styles.count}>{likesCount}</Text>
        </TouchableOpacity>
        <TouchableOpacity style={styles.iconBtn}>
          <Text style={{fontSize:18}}>💬</Text>
          <Text style={styles.count}>{item.commentsCount || 0}</Text>
        </TouchableOpacity>
        <TouchableOpacity style={styles.iconBtn}>
          <Text style={{fontSize:18}}>🔗</Text>
          <Text style={styles.count}>Share</Text>
        </TouchableOpacity>
      </View>
      <View style={styles.bottomLeft}>
        <Text style={styles.userText}>@{item.userName || "anon"}</Text>
        <Text style={styles.desc}>{item.caption}</Text>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    height: WINDOW_HEIGHT,
    backgroundColor: "black",
  },
  video: {
    width: "100%",
    height: "100%",
    position: "absolute",
  },
  rightColumn: {
    position: "absolute",
    right: 10,
    bottom: 120,
    alignItems: "center",
  },
  iconBtn: {
    marginBottom: 20,
    alignItems: "center",
  },
  count: {
    color: "white",
    marginTop: 6,
    fontSize: 12,
  },
  bottomLeft: {
    position: "absolute",
    left: 10,
    bottom: 30,
    width: "70%",
  },
  userText: {
    color: "white",
    fontWeight: "700",
    marginBottom: 6,
  },
  desc: {
    color: "white",
  },
});
// src/screens/FeedScreen.js
import React, { useEffect, useRef, useState } from "react";
import { View, FlatList, StyleSheet, StatusBar, ActivityIndicator } from "react-native";
import VideoItem from "../components/VideoItem";
import { collection, query, orderBy, onSnapshot } from "firebase/firestore";
import { db } from "../../firebase";

export default function FeedScreen() {
  const [videos, setVideos] = useState([]);
  const [loading, setLoading] = useState(true);
  const viewabilityConfig = { itemVisiblePercentThreshold: 80 };
  const [activeIndex, setActiveIndex] = useState(0);
  const onViewRef = useRef(({ changed }) => {
    changed.forEach(v => {
      if (v.isViewable) {
        setActiveIndex(v.index);
      }
    });
  });

  useEffect(() => {
    const q = query(collection(db, "videos"), orderBy("createdAt", "desc"));
    const unsub = onSnapshot(q, snapshot => {
      const arr = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
      setVideos(arr);
      setLoading(false);
    }, (err)=> {
      console.warn("feed snapshot err", err);
      setLoading(false);
    });
    return unsub;
  }, []);

  if (loading) {
    return (
      <View style={styles.loading}>
        <ActivityIndicator size="large" />
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <StatusBar hidden />
      <FlatList
        data={videos}
        keyExtractor={(i) => i.id}
        pagingEnabled
        snapToAlignment="start"
        decelerationRate="fast"
        renderItem={({ item, index }) => (
          <VideoItem item={item} isActive={index === activeIndex} />
        )}
        onViewableItemsChanged={onViewRef.current}
        viewabilityConfig={viewabilityConfig}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: "black" },
  loading: { flex: 1, alignItems: "center", justifyContent: "center" },
});
// src/screens/UploadScreen.js
import React, { useState } from "react";
import { View, Button, TextInput, StyleSheet, Alert, ActivityIndicator } from "react-native";
import * as ImagePicker from "expo-image-picker";
import { ref, uploadBytes, getDownloadURL } from "firebase/storage";
import { storage, db, auth } from "../../firebase";
import { collection, addDoc, serverTimestamp } from "firebase/firestore";

export default function UploadScreen() {
  const [caption, setCaption] = useState("");
  const [uploading, setUploading] = useState(false);
  const [videoUri, setVideoUri] = useState(null);

  const pickVideo = async () => {
    const perm = await ImagePicker.requestMediaLibraryPermissionsAsync();
    if (!perm.granted) {
      Alert.alert("Permission required", "Please allow media library access.");
      return;
    }
    const res = await ImagePicker.launchImageLibraryAsync({
      mediaTypes: ImagePicker.MediaTypeOptions.Videos,
      allowsEditing: false,
    });
    if (!res.cancelled) {
      setVideoUri(res.uri);
    }
  };

  const upload = async () => {
    if (!videoUri) {
      Alert.alert("Select a video first");
      return;
    }
    setUploading(true);
    try {
      const response = await fetch(videoUri);
      const blob = await response.blob();
      const filename = `videos/${Date.now()}_${Math.random().toString(36).slice(2,9)}.mp4`;
      const storageRef = ref(storage, filename);
      await uploadBytes(storageRef, blob);
      const url = await getDownloadURL(storageRef);

      await addDoc(collection(db, "videos"), {
        videoURL: url,
        caption: caption || "",
        userId: auth.currentUser?.uid || null,
        userName: `user_${(auth.currentUser?.uid || '').slice(0,5)}`,
        likes: [],
        commentsCount: 0,
        createdAt: serverTimestamp()
      });

      Alert.alert("Uploaded!", "Your video is now live on FOFO.");
      setCaption("");
      setVideoUri(null);
    } catch (e) {
      console.warn("upload err", e);
      Alert.alert("Upload failed", e.message);
    } finally {
      setUploading(false);
    }
  };

  return (
    <View style={styles.container}>
      <Button title="Pick video from gallery" onPress={pickVideo} />
      <TextInput
        placeholder="Write a caption..."
        value={caption}
        onChangeText={setCaption}
        style={styles.input}
      />
      {uploading ? <ActivityIndicator size="large" /> : <Button title="Upload to FOFO" onPress={upload} />}
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16, justifyContent: "center" },
  input: { borderWidth: 1, borderColor: "#ccc", padding: 10, marginVertical: 12, borderRadius: 6 }
});
// src/screens/ProfileScreen.js
import React, { useEffect, useState } from "react";
import { View, Text, FlatList, StyleSheet, TouchableOpacity } from "react-native";
import { collection, query, where, onSnapshot, orderBy } from "firebase/firestore";
import { db, auth } from "../../firebase";
import VideoItem from "../components/VideoItem";

export default function ProfileScreen() {
  const [videos, setVideos] = useState([]);

  useEffect(() => {
    const q = query(
      collection(db, "videos"),
      where("userId", "==", auth.currentUser?.uid || null),
      orderBy("createdAt", "desc")
    );
    const unsub = onSnapshot(q, snap => {
      setVideos(snap.docs.map(d => ({ id: d.id, ...d.data() })));
    }, (err)=>{ console.warn(err); });
    return unsub;
  }, []);

  return (
    <View style={styles.container}>
      <Text style={styles.header}>My FOFO Profile</Text>
      <FlatList
        data={videos}
        keyExtractor={(i)=>i.id}
        renderItem={({item, index}) => <VideoItem item={item} isActive={false} />}
        ListEmptyComponent={<Text>No uploads yet</Text>}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, paddingTop: 40 },
  header: { fontSize: 20, fontWeight: "700", padding: 12 }
});
{
  "expo": {
    "name": "FOFO",
    "slug": "fofo",
    "version": "1.0.0",
    "android": {
      "package": "com.yourcompany.fofo"
    },
    "platforms": ["android"],
    "sdkVersion": "48.0.0"
  }
}
{
  "expo": {
    "name": "FOFO",
    "slug": "fofo",
    "version": "1.0.0",
    "android": {
      "package": "com.yourcompany.fofo"
    },
    "platforms": ["android"],
    "sdkVersion": "48.0.0"
  }
}
