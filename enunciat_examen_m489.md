# EXAMEN · MÒDUL 489

## Programació de Dispositius Mòbils i Multimèdia

**Unitats Formatives:** RA2 i RA3  
**Curs:** 2n DAM · Videojocs  
**Data:** 23/03/2026  
**Durada:** 2 hores  

**Alumne/a:** diego salmeron angosto 
**Grup:** 2n dam  

---

## Posada en marxa de l'entorn

Consulta el fitxer **`README.md`** del projecte per a les instruccions completes d'instal·lació i arrencada (Node.js, servidor mock i Flutter).

---

> **Instruccions generals**
>
> - Respon cada pregunta en l'espai indicat (substitueix el text `[Escriu la teva resposta aquí]`).
> - Per a la part de codi, escriu directament en bloc `dart`. No és necessari que el codi compili, però ha de reflectir coneixement real de Flutter/Dart.
> - Tens el codi dels projectes **Cars** i **Phone** com a referència en el teu ordinador. **No pots accedir a internet** durant l'examen.
> - Desa el fitxer i lliura'l amb el nom: `EXAMEN_M489_[el_teu_nom].md`
> - Fes commit i push del .md modificat i de tots els arxius que hagis modificat

---

## BLOC 1 · ARQUITECTURA I CICLE DE VIDA *(RA 2)*

### Pregunta 1.1 – Comunicació entre Widgets *(12 punts)*

Al projecte **Cars**, el widget `CarsPage` gestiona el número de pàgina actual (`_currentPage`) i el passa a `CarsList1`. El widget `ButtonPanel` conté els botons "Anterior" i "Següent".

**a)** A `cars_page.dart`, el widget utilitza el mètode `setState` per gestionar la paginació. Explica:

- Quina és la funció de `setState` i per què cridar-lo fa que la UI es torni a dibuixar.
- Per quin motiu `_loadPage()` fa servir dos crides a `setState` separades (una a l'inici i una al final) en lloc d'una sola.

**Resposta:**

```
La funcion de setSate es el encargado de cambiar los estados de la ui para que se vuelva a dibujar y la razon por la cual _loadPage hace dos llamadas al setState se debe a que la primera llamada lo hace cuando esta el projecto esperando la respuesta de la API para mostrarun boton de carga y la segunda vez lo hace para quitar el boton de carga una vez reciba los datos ya sea una repuesta valida o de error
```

---

### Pregunta 1.2 – Cicle de vida d'un widget amb recursos *(13 punts)*

Al projecte **Camera**, el widget `CameraScreen` utilitza un `CameraController` per gestionar la càmera del dispositiu. Aquest controlador ocupa recursos del sistema (càmera, memòria) i cal alliberar-los correctament.

**a)** Quin mètode del cicle de vida de `State` s'usa a `CameraScreen` per alliberar el `CameraController` quan el widget és destruït? Escriu com es fa i explica per quina raó és imprescindible cridar-lo.

**Resposta:**

```
_takePicture es el que se encarga de tomar la foto pero no hay ningun metodo de camara que se encargue de matar el proceso de manera literal porque siempre esta activa 

1 Pero si se tiene en cuenta de que pueda dar un error en el spanchop entonces si que es el encargado de cargarse el camaraControler ya que no se ejecuta al estar cargando por el error captura al no poder conseguir acceder a la camara 

2 impidiendo que el projecto explote al no poder ejecutarse la camara en ningun momento
```

---

**b)** El `CameraController` s'inicialitza de forma asíncrona a `initState()` i el resultat es guarda a `_initializeControllerFuture`. Respon les preguntes següents:

- Per quin motiu no es pot fer `await` directament a `initState()`?
- Quina millora aporta a l'usuari usar `FutureBuilder` en lloc de bloquejar el fil?
- Com treballen junts `_initializeControllerFuture` i `FutureBuilder`?

**Resposta:**

```
1 porque tiene que iniciar todo el estado y solicitar permisos y dibujar toda la ui 
2 permite que este esperando una respuesta mientras va haciendo otras cosas sin necesidad de un await
3 se encargan de esperar a que el controlador de la camara se pueda activar y una vez conseguido pueda el builder recibir
ver la camara para presentarla en la aplicacion
```

---

## BLOC 2 · COMUNICACIÓ, PERSISTÈNCIA I PROVES *(RA 2 — 35 punts)*

### Pregunta 2.1 – Consum d'API i robustesa *(18 punts)*

Analitza el mètode `getCarsPage(int page, int limit)` de `car_http_service.dart`.

Què passaria si el servidor de l'API trigués 60 segons a respondre? L'aplicació quedaria bloquejada per a l'usuari? Per què? Escriu com implementaries un *timeout* de 10 segons a la petició HTTP.

**Resposta:**
1 La aplicacion estaria esperando a que hubiera una respuesta del metodo mostrando el circulo de carga del _loadPage 
esperando a que de una respuesta el service con la lista de coches para devolver la lista o el error
2 debido a que el programa estaria esperando una respuesta 

```dart
// Escriu la modificació al getCarsPage aquí:
  Future<List<CarsModel>> getCarsPage(int page, int limit) async {
    final offset = (page - 1) * limit;
    final uri = _buildUri('/v1/cars', {'limit': '$limit', 'offset': '$offset'});

    final response = await http
        .get(uri, headers: _headers)
        // parte que se encarga para que la duracion maxima de la espera sean 10 segundos
        // una vez termina el timeout devuelve un null al no conseguir respuesta
        .timeout(const Duration(seconds: 10)); 

    if (response.statusCode == 200) { // provocando que mande un mensaje de error al no recibir una respuesta de la API
      return CarsModel.listFromJsonString(response.body);
    } else {
      throw Exception('Error ${response.statusCode}: ${response.body}');
    }
  }
```

---

### Pregunta 2.2 – Models de dades  *(17 punts)*

Analitza el constructor `factory CarsModel.fromMapToCarObject(Map<String, dynamic> json)` de `car_model.dart`.

**a)** Imagina que l'API retorna per error el camp `year` com a `String` en lloc d'`int` (per exemple, `"2021"` en lloc de `2021`). El codi actual fallaria. Escriu com resoldries el problema.

**Resposta:**

```
Cambiaria en el model el indentificar de `as int` a `as String?` o si no tambien miraria en caso de necesidad para no cambiar todo el codigo podria una vez recibido el string convertilo y pasarlo a un int si cumple con las condiciones si no seria adaptar todo el codigo para que year sea detectado como string

```

```dart
  factory CarsModel.fromMapToCarObject(Map<String, dynamic> json) {
    return CarsModel(
      id: json['id'] as int,
      year: json['year'] as String?,
      make: json['make'] as String? ?? '',
    );
  }
```

---

**b)** Al fitxer `class_model_test.dart`, el test utilitza un `const jsonString` amb un JSON escrit a mà en lloc de fer una petició real a l'API de RapidAPI. Explica per quin motiu és millor simular el JSON en un test unitari.

**Resposta:**

```
Es mejor para tener una respuesta rapida sin depender del servidor y tener que realizar las validaciones de la api para comprobar que los valores sean correctos y tengan su valor bien designados como string o int a la hora de importar
```

---

## BLOC 3 · IMPLEMENTACIÓ PRÀCTICA *(RA 3 — 30 punts)*

### Exercici – Widget de detall amb dades remotes

Imagina que volem crear una pantalla de detall per a cada cotxe del projecte Cars. Implementa el mètode `build` d'un widget `StatelessWidget` anomenat `CarDetailPage` que compleixi els requisits següents:

1. Rep un paràmetre `final CarsModel car` al constructor.
2. Mostra el **make** i el **model** del cotxe com a títol destacat (`Text` amb estil gran i negreta).
3. Mostra una **icona diferent** depenent del `type` del cotxe:
   - Si el `type` és `'SUV'`, mostra `Icons.directions_car`.
   - Per qualsevol altre tipus, mostra `Icons.car_rental`.
4. Afegeix un botó `ElevatedButton` que, quan es premi, mostri un `SnackBar` amb el text: `"Cotxe seleccionat: [make] [model]"`.

```dart
// Escriu el teu codi aquí:

class CarDetailPage extends StatelessWidget {
  final CarsModel car;

  const CarDetailPage({super.key, required this.car});

  @override
  Widget build(BuildContext context) {

      return ListView.builder(
      itemCount: cars.length,
      itemBuilder: (context, index) {
        final car = cars[index];
        return ListTile(
          leading: const Icon(Icons.directions_car),
          title: Text('${car.make} ${car.model}', 
            Style( //dentro del text indicar el color y el tamaño mediante el style
              color.text: Colors(black), 
              Size 20, 
              color.text.style(bold)
            ),
          ) 
          
          /*segun el tipo de modelo que tenga el coche mediante el su tipo se tiene una imagen a su nombre y cuando reciba el typo de coche que es automaticamente seleciona la imagen si es "suv" mostrara la imagen suv.png y o bmv 
          bmv.png
          */
          Image: Image("assets/images/${car.type}.png"),

          // para poner una imagen especifica solo si es suv
          if (${car.type}.equals("SUV"))
            Image: Image("assets/images/suv.png"),
          else{
             Image: Image("assets/images/cualquiera.png"),
          };

          ElevateButton(
            Text("Selecionar"),
            //snake bar para mostrar el coche
            _snakebar(),
          ),
        ),
      };
    ),
  }

  class _snakebar(){
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text('Coche selecionado: ${car.make} ${car.model}'), // muestra la informacion durante 
        duration: const Duration(seconds: 10),
      ),
    )
  }
}
```

---

**Ampliació (nivell Expert):** Afegeix un `FutureBuilder` que cridi al mètode `CarHttpService().getCarsPage(1, 5)` i mentre espera les dades mostri un `CircularProgressIndicator`. Quan les dades estiguin llestes, mostra un `ListView.builder` amb el `make` de cada cotxe. Si hi hagués un error, mostra un `Text` en color vermell amb el missatge de l'error.

```dart

Future<void> _loadPage() async { // 
    setState(() {
      _isLoading = true; // muestra carga
      _error = null; // marca error como null
    });
    try {
      final cars = await _service.getCarsPage(1, 5); // llama al cars service especificado a 1, 5
      setState(() {
        _cars = cars; // consigue los coches
        _isLoading = false; // para la rueda de carga
      });
    } catch (e) {
      setState(() {
        _error = e.toString(); // recoge el error
        _isLoading = false; // para la carga
      });
    }
  }


FutureBuilder futureMake {
  if (isLoading) { // si esta cargando en true muestra el circulo cargando
      return const Center(child: CircularProgressIndicator());
    }


    if (error != null) { // si se ha producido un error lo muestra
      return Center(
        child: Text('Error: $error', style: const TextStyle(color: Colors.red)), // lo devuelve de color rojo
      );
    }


    if (cars.isEmpty) { // lista vacia
      return const Center(child: Text('No hi ha més cotxes.'));
    }

    // Estat 4: Dades rebudes correctament
    return ListView.builder(
      itemCount: cars.length,
      itemBuilder: (context, index) {
        final car = cars[index];
        return ListTile(
          leading: const Icon(Icons.directions_car),
          title: Text('${car.make}'), // muestra los makes de cada coche
          trailing: Text(
            car.color,
            style: const TextStyle(color: Colors.blueGrey),
          ),
        );
      },
    );
}  

```

---

## BLOC 4 · EXTENSIÓ DEL SERVEI HTTP *(RA 2 — 10 punts)*

### Exercici 4.1 – Mètode parametritzat a `CarHttpService` *(10 punts)*

El servidor mock local té disponible un  endpoint de cerca:

```
GET http://localhost:8080/v1/cars/search?make=Toyota&model=Corolla
```

- El paràmetre `make` filtra per marca (coincidència parcial, insensible a majúscules).
- El paràmetre `model` filtra per model (coincidència parcial, insensible a majúscules).
- Tots dos paràmetres són opcionals: si no s'envien, retorna tots els cotxes.

Exemples vàlids:

- `/v1/cars/search?make=Toyota` → tots els Toyota
- `/v1/cars/search?model=X5` → tots els cotxes amb "X5" al model
- `/v1/cars/search?make=BMW&model=X` → BMW amb "X" al model

**Implementa** el mètode `getCarsByFilter` a la classe `CarHttpService` existent, seguint el mateix patrons que `getCarsPage`:

```dart

    [make] // conseguir del page el make para llamar al siguiente metodo y conseguir solo por el make
    Future<List<CarsModel>> getCarsFiltrerByMake(int page, int limit) async {
    final offset = (page - 1) * limit;
    final uri = _buildUri('/v1/cars/search?make=$make', {'limit': '$limit', 'offset': '$offset'});

    final response = await http
        .get(uri, headers: _headers)
        .timeout(const Duration(seconds: 10));

    if (response.statusCode == 200) {
      return CarsModel.listFromJsonString(response.body);
    } else {
      throw Exception('Error ${response.statusCode}: ${response.body}');
    }
  }

    [model] // conseguir del page el model para realizar la consulta de filtrar solo por modelo
    Future<List<CarsModel>> getCarsFiltrerByModel(int page, int limit) async {
    final offset = (page - 1) * limit;
    final uri = _buildUri('/v1/cars/search?model=$model', {'limit': '$limit', 'offset': '$offset'});

    final response = await http
        .get(uri, headers: _headers)
        .timeout(const Duration(seconds: 10));

    if (response.statusCode == 200) {
      return CarsModel.listFromJsonString(response.body);
    } else {
      throw Exception('Error ${response.statusCode}: ${response.body}');
    }
  }

    // para realizar la consulta de ambos
    Future<List<CarsModel>> getCarsFiltrerByMakeAndModel(int page, int limit) async {
    final offset = (page - 1) * limit;
    final uri = _buildUri('/v1/cars/search?make=${make}&model=${model}', {'limit': '$limit', 'offset': '$offset'});

    final response = await http
        .get(uri, headers: _headers)
        .timeout(const Duration(seconds: 10));

    if (response.statusCode == 200) {
      return CarsModel.listFromJsonString(response.body);
    } else {
      throw Exception('Error ${response.statusCode}: ${response.body}');
    }
  }

    // para realizar la consulta de todo de manera opcional 
    Future<List<CarsModel>> getCarsFiltrerBy(int page, int limit) async {
    final offset = (page - 1) * limit;
    final uri = _buildUri('/v1/cars/', {'make=': '$make', 'model=': '$model', 'limit': '$limit', 'offset': '$offset'});

    final response = await http
        .get(uri, headers: _headers)
        .timeout(const Duration(seconds: 10));

    if (response.statusCode == 200) {
      return CarsModel.listFromJsonString(response.body);
    } else {
      throw Exception('Error ${response.statusCode}: ${response.body}');
    }
  }
```

Requisits:

1. Utilitza el mètode privat `_buildUri(String path, Map<String, String> queryParams)` ja existent.
2. Només afegeix els paràmetres `make` i/o `model` al mapa si el valor no és `null` ni buit (`isEmpty`).
3. Gestiona errors i timeout amb el mateix mecanisme que `getCarsPage`.

**Resposta:**

```dart


```

---

## Resum de l'examen

| Bloc | RA | Punts màxims |
|------|----|:------------:|
| Bloc 1 – Arquitectura i Cicle de vida | RA 2 | 25 |
| Bloc 2 – Comunicació, Persistència i Proves | RA 2  | 35 |
| Bloc 3 – `CarDetailPage` (base) | RA 3 | 20 |
| Bloc 3 – Ampliació `FutureBuilder`  | RA 3 | 10 |
| Bloc 4 – Extensió del servei HTTP | RA 2 | 10 |
| **TOTAL** | | **100** |

---
-