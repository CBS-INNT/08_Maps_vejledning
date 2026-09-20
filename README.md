# 08 Maps vejledning
I dag skal vi arbejde med at få et `react-native-maps` til at virke på en simple app. Start med at læse dokumentationen her: 

https://docs.expo.dev/versions/latest/sdk/map-view/ <br>
https://github.com/react-native-maps/react-native-maps

## Opret app'en i VSC
1. Start med at oprette et nyt projekt som vi plejer: `npx create-expo-app --template blank 08_Maps`.
2. Husk at skrive `cd 08_Maps` bagefter inden du går videre.
3. Kopiere derefter følgende ind i din `package.json`:
```json
"dependencies": {
    "@expo/vector-icons": "^15.0.3",
    "@react-native-async-storage/async-storage": "2.2.0",
    "@react-navigation/bottom-tabs": "^7.4.7",
    "expo": "^57.0.24",
    "expo-constants": "~57.0.19",
    "expo-font": "~57.0.4",
    "expo-linear-gradient": "~57.0.2",
    "expo-location": "~57.0.19",
    "expo-status-bar": "~57.0.1",
    "react": "19.2.3",
    "react-native": "0.86.3",
    "react-native-maps": "1.27.2",
    "react-native-safe-area-context": "~5.7.0",
    "react-native-screens": "~4.26.0"
  },
```
4. Kør `npm install`

## style/GlobalStyles.js
Opret `GlobalStyles.js` og indsæt følgende kode:

```javascript
import { StyleSheet } from "react-native";

const GlobalStyles = {
  home: StyleSheet.create({
    container: {
      flex: 1,
      backgroundColor: "#fff",
      alignItems: "center",
      justifyContent: "center",
    },
    bcg: {
      padding: 80,
      alignItems: "center",
      width: "100%",
      height: "100%",
    },
    title: {
      fontSize: 32,
      fontWeight: "bold",
      color: "#333",
      marginBottom: 80,
    },
    input: {
      height: 40,
      borderColor: "lightgray",
      borderWidth: 1,
      marginBottom: 20,
      paddingHorizontal: 10,
      width: "90%",
      borderRadius: 20,
      backgroundColor: "#f9f9f9",
    },
    button: {
      backgroundColor: "#FF0000",
      padding: 10,
      borderRadius: 20,
      alignItems: "center",
      width: "90%",
      marginTop: 40,
    },
    buttonText: {
      color: "#FFFFFF",
      fontSize: 16,
      fontWeight: "bold",
    },
    logo: {
      width: 100,
      height: 100,
      marginBottom: 50,
      resizeMode: "contain",
    },
  }),

  map: StyleSheet.create({
    container: { flex: 1 },
    map: { flex: 1 },
    loadingContainer: {
      flex: 1,
      justifyContent: "center",
      alignItems: "center",
    },
  }),
};

export default GlobalStyles;
```

## App.js
Start med at lave en TapNavigator til de 2 screens vi skal bruge.

### Navigation
1. Lav to screens i mappen `screens`:
    - `Home.js`
    - `Map.js`

2. I hver screen indsæt følgende kode (husk at ændre navn osv.):
```javascript
import { View, Text } from 'react-native';
import GlobalStyles from "../style/GlobalStyles";

export default function ???({navigation}) {
  return (
    <View>
      <Text>This is ??? Screen</Text>
    </View>
  );
};
```
3. Gå tilbage til App.js, hvor vi nu skal lave vores navigation til disse skræme.
4. Importer `NavigationContainer`, `createBottomTabNavigator`, `SafeAreaProvider`, `Ionicons` og vores to skærme.
5. Lav en `createBottomTabNavigator();`
6. I din export funktion skal du lave en tab navigation som er wrapped i en `NavigationContainer` og en `SafeAreaProvider`
**Tip**

```javascript
export default function App() {
  return (
    <???>
      <???>
        <???.???
          screenOptions={({ route }) => ({
            // Farver for aktiv/inaktiv tab
            tabBarActiveTintColor: "#FF0000",
            tabBarInactiveTintColor: "gray",
            // Vælg ikon ud fra tab-navn
            tabBarIcon: ({ color, size }) => {
              const iconName = route.name === "Home" ? "home" : "map";
              return <Ionicons name={iconName} size={size} color={color} />;
            },
            headerShown: false, // skjul header
          })}
        >
          <???.??? name="Home" component={Home} />
          <???.??? name="Map" component={Map} />
        </Tab.Navigator>
      </NavigationContainer>
    </SafeAreaProvider>
  );
}
```
7. Check at din navigation virker og fortsæt til Home.js (teksten er nok oppe i venstre hjørne)

## Home.js
1. Start med at importere dine moduler
```javascript
import { Text, View, TextInput, TouchableOpacity, Image } from "react-native";
import { SafeAreaView } from "react-native-safe-area-context";
import AsyncStorage from "@react-native-async-storage/async-storage";
import { useState } from "react";
import GlobalStyles from "../style/GlobalStyles";
```

2. Vi skal nu lave 3 `const` med `useState` som det første i vores export function:
  - `latitude` med et tomt string 
  - `longitude` med et tomt string
  - `markers`med et tomt array

**Tip**
- `const [latitude, setLatitude] = useState('');`

3. Lav en funktion til at validere koordinater (skal være tal og inden for gyldigt område)

**Tip**
```javascript
  const isValidCoord = (lat, lon) =>
      !isNaN(???) &&
      !isNaN(???) &&
      ??? >= -90 &&
      ??? <= 90 &&
      ??? >= -180 &&
      ??? <= 180;
```
4. Vi skal nu lave en `addAndSaveMarker` funktion. Denne funktion tilføjer en ny markør til en liste over eksisterende markører og gemmer den opdaterede liste ved hjælp af `AsyncStorage`.
4.1. Start med at kontrollere inputtet
```javascript
 const addAndSaveMarker = async () => {
    const lat = parseFloat(???.replace(",", ".")); // konverter string → float
    const lon = parseFloat(???.replace(",", "."));
    if (!isValidCoord(lat, lon)) {
      ???("Ugyldige koordinater");
      return;
    }
```
4.1. Opret en ny markør
```javascript
  const newMarker = {
        id: Date.now().toString(),
        latitude: lat,
        longitude: lon,
        title: "Custom",
      };
```
4.2. Tilføj markøren til listen 
```javascript
  const updatedMarkers = [...markers, newMarker]; // tilføj til listen
      setMarkers(updatedMarkers);
      setLatitude("");
      setLongitude("");
```
4.3. Gem hele listen i AsyncStorage. Dette gemmer listen af markører permanent på enheden.
```javascript
try {
      // Gem hele listen af markører i AsyncStorage
      await AsyncStorage.setItem("markers", JSON.stringify(updatedMarkers));
      navigation.navigate("Map"); // skift til Map-tab
    } catch (error) {
      console.error("Error saving markers", error);
    }
```
Fedt det var hele `addAndSaveMarker` funktionen

5. Nu skal vi lave vores `return` statement
5.1. Først laver vi en konstant variabel (før `return`), der får tildelt værdien af GlobalStyles.home: `const styles = GlobalStyles.home;`
5.2. Download `logo.png` her fra repo og indsæt det i dine `assests`
5.3. Indsæt følgende i din `return`. Vi bruger `SafeAreaView` for at sikre at indholdet er indenfor skærmen.
```javascript
   return (
    <SafeAreaView style={[styles.container, { backgroundColor: "#fff" }]}>
      <View style={styles.bcg}>
        {/* Logo hentet fra assets */}
        <??? source={require("../assets/logo.png")} style={styles.logo} />

        {/* Titel */}
        <Text style={styles.title}>INNT Map App</Text>

        {/* Inputfelter til latitude/longitude */}
        <TextInput
          style={styles.input}
          value={latitude}
          onChangeText={setLatitude}
          placeholder="latitude (-90 to 90)"
          keyboardType="decimal-pad"
          returnKeyType="next"
        />
        <TextInput
          style={styles.input}
          value={longitude}
          onChangeText={setLongitude}
          placeholder="longitude (-180 to 180)"
          keyboardType="decimal-pad"
        />

        {/* Knap til at gemme og hoppe til kortet */}
        <??? style={styles.button} onPress={???}>
          <Text style={styles.buttonText}>Search</Text>
        </TouchableOpacity>
      </View>
    </???>
  );
```

Din Home.js skulle nu gerne være done.

## Map.js
I `Map.js` vil vi nu gerne skabe vores map. 
1. Start med imports:
```javascript
import * as Location from "expo-location";
import { useState, useCallback } from "react";
import { ActivityIndicator } from "react-native";
import { View } from "react-native";
import MapView, { Marker } from "react-native-maps";
import AsyncStorage from "@react-native-async-storage/async-storage";
import { useFocusEffect } from "@react-navigation/native";
import GlobalStyles from "../style/GlobalStyles";
```

2. Lav nu 3 `const` med `useState` i export funktionen
  - `markers` med tomt array
  - `loading` med `useState(true)`
  - `initialRegion` med 4 værdier `latitude:???`, `longitude:???`, `latitudeDelta: 0.0922`, `longitudeDelta: 0.0421`. Sæt dine longitude og latitude værdier til det som dit inital map skal vise - f.eks. KBH

3. Lav en konstant variabel, der får tildelt værdien af GlobalStyles.map
4. Hent markørerne fra AsyncStorage. Den skal bruge en `try - catch` funktionalitet som afventer vores `markers` fra vores `AsyncStorage`. 
```javascript
const getMarkers = async () => {
    ??? {
      const raw = await ???.getItem("markers");
      const list = raw ? JSON.parse(raw) : [];
      setMarkers(list);

      // Flyt kortet til den senest gemte markør
      const latest = list.at(-1);
      if (latest) {
        setRegion((r) => ({
          ...r,
          ???: latest.latitude,
          ???: latest.longitude,
        }));
      }
    } ??? (e) {
      console.error("Error retrieving markers", e);
    }
  };
```

5. (valgfrit) Det er også muligt at hente brugerens egen lokation med `getLocation`. Du skal lige give tilladelse til dette. 
```javascript
  const ??? = async () => {
    const { status } = ??? Location.requestForegroundPermissionsAsync();
    if (status !== "granted") return;
    const { coords } = await Location.getCurrentPositionAsync({});
    setRegion((r) => ({
      ...r,
      latitude: coords.latitude,
      longitude: coords.longitude,
    }));
  };
```

6. Lav en `useFocusEffect` funktion, der henter både markører og lokation parallelt, og slår loading fra, når de er færdige.
```javascript
  useFocusEffect(
    useCallback(() => {
      setLoading(true);
      Promise.all([getMarkers(), getLocation()]).finally(() =>
        setLoading(false)
      );
    }, [])
  );
```
Koden gør, at hver gang skærmen bliver vist, starter en "loading"-tilstand, henter både markører og lokation parallelt, og slår loading fra, når de er færdige

7. Lav en loader-indikation
```javascript
if (loading) {
    return (
      <View style={styles.loadingContainer}>
        <ActivityIndicator size="large" />
      </View>
    );
  }
```

8. Nu skal du lave din `return`, som viser et kort med brugerens position og alle gemte markører. Brugeren kan tilføje en ny markør ved at long-presse på kortet, hvorefter markøren gemmes både i appens state og i AsyncStorage. Kortets position styres af region, som opdateres, når brugeren flytter rundt på kortet.
```javascript
return (
    <View style={styles.container}>
      <MapView
        style={styles.map}
        region={region} // styrer kortets position
        onRegionChangeComplete={setRegion}
        showsUserLocation
        // Mulighed: long-press for at tilføje ny markør direkte på kortet
        onLongPress={(e) => {
          const { latitude, longitude } = e.nativeEvent.coordinate;
          const ??? = {
            id: Date.now().toString(),
            latitude,
            longitude,
            title: "Drop pin",
          };
          const next = [...markers, newMarker];
          setMarkers(next);
          AsyncStorage.setItem("markers", JSON.stringify(next));
        }}
      >
        {/* Tegn alle markører på kortet */}
        {markers.map((m) => (
          <Marker
            key={m.id ?? `${m.latitude},${m.longitude}`}
            ???={{ latitude: m.latitude, longitude: m.longitude }}
            title={m.title ?? "Marker"}
            tracksViewChanges={false}
            pinColor="#FF0000"
          />
        ))}
      </???>
    </View>
  );
```
Årsagen til at vi ikke bruger `SafeAreaView` her i Map.js er fordi vi ønsker at kortet skal gå helt ud til kanterne. 

Nu virker din Map.js også, og hele app'en skulle gerne fungerer. 
