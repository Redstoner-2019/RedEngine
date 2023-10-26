# **Game Pipeline**
Pipeline Version: 1.0
Dokumentation Stand: 26.10.2023 10:14
## 1 Getting started
### 1.1 Ein Fenster Erstellen

Um ein Fenster zu erstellen muss nur ein Camera Objekt erstellt und mit ```.start()``` gestartet werden.
Daraufhin wird sich ein Fenster mit der Größe des Monitors öffnen.

Ein Beispiel:
```java 
Camera camera = new Camera(new ArrayList<>());
camera.start();
```
Wie man bereits sieht wird bei der Erstellung des Camera-Objekts eine Liste mit übergeben. Diese Liste ist eine Liste des Typs ```String```.
Diese Liste sind die Texturen die direkt nach dem Start des Fensters geladen werden. Wenn als Beispiel
```java 
Camera camera = new Camera(Arrays.asList("grass.png"));
```
verwendet wird, dann wird die Textur ```grass.png``` in dem Ordner ```resources/textures/``` geladen. \

**WICHTIG!**

Texturen müssen ZWINGEND im PNG format sein, ansonsten können diese nicht geladen werden!

Es ist allerdings auch möglich einfach eine leere Liste zu übergeben (Nicht null!), dabei werden dann keine Texturen direkt geladen.\
Wenn man den obrigen Code so ausführt gibt es aktuell jetzt noch das Problem dass das Fenster einfach Weiß ist und es die Meldung ```Keine Rückmeldung!``` gibt. Dafür muss eine Render-loop
erstellt werden. Dies ist einfach zu tun.\

Dafür können wir einfach
```java
List<GameObjectData> objects = new ArrayList<>();
List<Shader> shaders = new ArrayList<>();

while (!camera.shouldWindowClose()) {
    camera.render(objects, shaders);
}
```
verwenden. Was tut dieser Code jetzt allerdings?\
Zunächst werden 2 Listen erstellt. Eine Liste vom Typ ``GameObjectData`` und eine Liste vom Typ ``Shader``.
Die Namen dieser Listen sind irrelevant, es sind nur die Datentypen wichtig. In diesem Fall in der ``objects``-Liste werden
die Objekte welche Gerendert werden sollen an die Kamera übergeben und gerendert. Hierbei ist zu beachten, dass das erste Element der Liste als erstes gerendert wird
und somit von später gerenderten Objekten überlagert werden kann!\
Dieser Code reicht jetzt aber erst einmal aus um ein Fenster zu erzeugen.

### 1.2 Das Fenster einstellen

Als nächtes kommen wir zu nützlichen Dingen die man an dem Fenster einstellen kann.

Zuallererst wollen wir uns die Framerate (Also FPS) im Fenstertitel anzeigen lassen. Das ändern des Fenstertitels ist
mit dem folgenden Code einfach getan:

```java
camera.setTitle("Hello World!");
```
Wenn wir diesen Code in die ```while```-schleife einfügen wird sich der Fenstertitel auf ``Hello World!`` ändern.
Wir wollen aber die Bildrate anzeigen lassen. Dafür können wir die Methode ```camera.getFps() ``` verwenden. Wenn wir jetzt den Code auf z.B.
```java
camera.setTitle(camera.getFps() + " FPS");
```
ändern wird uns jetzt im Fenstertitel die Bildrate angezeigt!

Der Komplette Code bisher sollte nun etwa so aussehen:

```java
public static void main(String[] args){
    Camera camera = new Camera(new ArrayList<>());
    camera.start();
    
    List<GameObjectData> objects = new ArrayList<>();
    List<Shader> shaders = new ArrayList<>();
    
    while (!camera.shouldWindowClose()) {
        camera.render(objects, shaders);
        camera.setTitle(camera.getFps() + " FPS");
    }
}
```

### 1.3 Rendern von Texturen
Allerdings ist ein schwarzer Bildschirm nicht sonderlich interessant. Daher kommen
wir jetzt zum wichtigen Thema, wie kann man Texturen auf den Bildschirm rendern?\
Dafür muss die Textur zunächst geladen werden. Dafür kann man einfach ein neues Texture Objekt
erzeugen. Dazu muss eine URL angegeben werden wo sich das Objekt befindet. Dort kann man z.B. wie in diesem Beispiel
mit ```Resources.getResource("textures/texture.png")``` eine URL zu der Textur Datei erzeugt werden, welche sich in diesem
Fall im ```resources/textures/``` Ordner befindet. Diese URL kann an das Objekt weitergegeben werden.
Der Code dafür sieht folgendermaßen aus:
```java
Texture texture = new Texture(Resources.getResource("textures/skyTexture.png"));
```
Diese Textur kann nun der Kamera zugewiesen werden, damit die Kamera weiß wo sich die Textur befindet:
```java
Texture texture = new Texture(Resources.getResource("textures/skyTexture.png"));
camera.addTexture(texture,"sky");
```
Das zweite Argument in diesem Fall ist der "Pfad" an dem sich die Textur befindet. Dieser muss
in einem GameObject angegeben werden damit die Kamera weiß welche Textur auf das GameObject angewendet werden soll.\
Ein GameObject kann folgendermaßen erzeugt werden:
```java
GameObject object = new GameObject(new Vector2(0, 0),"sky");
```
In diesem Fall sehen wir direkt, dass wir 2 Argumente für das GameObject beötigen.
Das erste ist die Position des GameObjects. Dieses wird als Vector2 angegeben, in diesem Fall befindet sich das Object bei 0 / 0.\
Das zweite Argument ist die Textur des Objekts. Diese kann Nachträglich noch geändert werden, muss aber vorher der Kamera als referenz wie im vorherigen Schritt zugewiesen werden!
Dieses sollte VOR der while-schleife erzeugt werden und der ``objects``-Liste von vorhin hinzugefügt werden.
Wenn wir dies nun tun wird dieses Objekt auf dem Bildschirm erscheinen.

Der volle Code sieht nun so aus:

```java
public static void main(String[] args){
    Camera camera = new Camera(new ArrayList<>());
    camera.start();

    Texture texture = new Texture(Resources.getResource("textures/skyTexture.png"));
    camera.addTexture(texture,"sky");

    GameObject object = new GameObject(new Vector2(0, 0),"sky");

    List<GameObjectData> objects = new ArrayList<>();
    List<Shader> shaders = new ArrayList<>();

    objects.add(object);

    while (!camera.shouldWindowClose()) {
        camera.render(objects, shaders);
        camera.setTitle(camera.getFps() + " FPS");
    }
}
```

## 2 Input
### 2.1 Keyboard Listener
Um Tastatureingaben auszulesen muss lediglich ein KeyboardListener erstellt werden.
```java
KeyboadListener keyboadListener = new KeyboadListener(camera);
```
Dieser muss bei der Erzeugung der Kamera zugewiesen werden.

Jetzt kann innerhalb der Gameloop mithilfe von z.B.
```java
if(keyboadListener.isKeyDown(GLFW.GLFW_KEY_W)){
    System.out.println("Hello World!");
}
```
``Hello World!`` ausgeben, SOLANGE die Taste ``W`` gedrückt ist.

Hier gibt es allerdings unterschiede in den Funktionen des Listeners.

_isKeyDown()_

Diese Methode prüft ob die Taste AKTUELL gedrückt ist.

_isKeyPressed()_

Diese Methode prüft ob die Taste gedrückt wurde. Sie gibt nur 1 Mal ``true`` zurück bis die Taste erneut gedrückt wird.

_isKeyReleased()_

Das gleiche wie bei ``isKeyPressed()``, allerdings beim loslassen der Taste.

### 2.2 Mouse Listener

Der MouseListener kann die Position des Mauszeigers abrufen. Der MouseListener kann **NICHT** prüfen ob eine Maustaste gedrückt ist, dafür wird ein MouseButtonListener benötigt.

Ein MouseListener kann folgendermaßen erzeugt werden:
```java
MouseListener mouseListener = new MouseListener(camera);
```

Jezt kann man mithilfe der Methode ``.getLocation()`` kann jetzt ein ``Vector2`` von der Position des Mauszeigers
erhalten werden.

### 2.3 Mouse Button Listener
Mit einem Mosuse Button Listener kann man auf interaktion von Tasten auf der Maus Prüfen.
Er kann folgendermaßen erzeugt werden:
```java
MouseButtonListener mouseButtonListener = new MouseButtonListener(camera);
```
Dieser listener hat die Methode ``isKeyDown()``, diese gibt ``true`` zurück, solange die Taste gedrückt ist.
