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
      "@react-native-async-storage/async-storage": "2.2.0",
      "@react-navigation/bottom-tabs": "^7.4.7",
      "expo": "~54.0.10",
      "expo-constants": "~18.0.9",
      "expo-linear-gradient": "~15.0.7",
      "expo-location": "~19.0.7",
      "expo-status-bar": "~3.0.8",
      "react": "19.1.0",
      "react-native": "0.81.4",
      "react-native-maps": "1.20.1",
      "react-native-vector-icons": "^10.3.0"
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
          onChangeText={???}
          placeholder="???"
          keyboardType="decimal-pad"
          returnKeyType="next"
        />
        <TextInput
          style={styles.input}
          value={longitude}
          onChangeText={???}
          placeholder="???"
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
Din `Home.js` skulle nu gerne være done.
