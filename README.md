# WFormularios

This project was generated with [Angular CLI](https://github.com/angular/angular-cli) version 16.2.5.

## Tutorial

### Workspace
Generamos un workspace vacío con el comando `ng new --create-application false w-formularios`, donde el parámetro `--create-application false` indica que no se va a crear un proyecto automáticamente en src/app y `w-formularios` es el nombre del workspace con el cual se crea la carpeta y el `package.json`.

### Proyecto

Dentro del workspace (`cd w-formularios`) vamos a generar una app de angular con el comando `ng g app web-reserva`. El comando `ng g app` es el atajo equivalente a `ng generate application` y el parámetro `web-reserva` es el nombre del proyecto.

De forma interactiva pide si usar el routing de Angular y el lenguaje para los estilos. Si no indicamos nada por defecto no se usa routing y los estilos son CSS.

### Componente

Creamos el componente del formulario con `ng g c form-reserva`. El comando `ng g c` es un atajo para `ng generate component` y `form-reserva` es el nombre del componente.

En projects/web-reserva/src/app/ se ha creado `form-reserva/` con el html,css y ts del componente. En el html creamos un formulario con los datos necesarios.

Arrancamos el proyecto con `npm run start` y ya lo deberiamos ver en `localhost:4200`. El comando `npm run start` es equivalente a `ng serve` debido a la configuración por defecto del `package.json`.

En `app.component.html` eliminamos el contenido de ejemplo y creamos el elemento del formulario ya creado `<app-form-reserva>`. Modificamos el `form-reserva.component.html` con todos los datos necesarios para el formulario.

Para hacer el formulario reactivo y asi poder trabajar con los datos del formulario importamos ReactiveFormsModule en el app.module.ts y asi se pueda usar en los `.html`. En el `form-reserva.component.ts` creamos la propiedad `formulario` como un FormGroup. Se asigna el `<form>` al formulario con `[formGroup]='formulario'` y cada `<input>` se corresponde a un `FormControl` con el nombre que le indicamos en `formControlName`. Obtenemos los valores con `this.formulario.value` pero observaremos que los parametros que todavía no hemos definido como FormControl ni hemos asignado el `formControlName` son ignorados.

Los datos obtenidos del formulario se pueden guardar como una interfaz.

### API sencilla

Para evitar desarrollar una API podemos crear una sencilla con `npx json-server datos.json`. Se instalará `json-server` y se iniciará en `localhost:3000`. El parametro `datos.json` indica el fichero donde se guardará toda la base de datos. En ese json podemos definir los datos con los que queremos trabajar, en este caso le agregamos `"reserva":[]` al final.

El funcionamiento es sencillo: si hacemos `POST /reserva` se insertará un objeto con los datos JSON que mandamos, si hacemos `PATCH /reserva/:id` se modifican los campos que le indiquemos, si hacemos `PUT /reserva/:id` se sustituye el objeto y si hacemos `DELETE /reserva/:id` se elimina. Con `GET /reserva` obtenemos el listado completo o con `GET /reserva/:id` obtenemos un objeto.

Creamos el servicio `ng g s api` o `ng generate service api` para crear el fichero `api.service.ts`. En el constructor obtenemos un `HttpClient` que usaremos para la API. Para que funcione hay que importar el `HttpClientModule` en el `app.module.ts`. En `api.service.ts` definimos los métodos que interactúan con la API. En este caso tambien creare un servicio `reserva.service.ts` que se encarga de interactuar con esa API, pero con un solo servicio se puede hacer perfectamente.

También se puede crear un componente para ver mis reservas `ng g c list-reserva`, en el cual utilizaremos la directiva *ngFor para listar todas las reservas, las cuales hemos cargado utilizando el mismo servicio de la API.

Además podemos crear un componente para editar la reserva `ng g c edit-reserva`, en el cual tal vez no queremos que se encuentren todos los campos sino solo algunos. El formulario será más sencillo. Utilizaremos @Input y @Output para notificar al componente edit-reserva de cual reserva hemos seleccionado desde el componente list-reserva. Luego simplemente utilizaremos la API de patch para actualizar los campos modificados.

### Validación de formularios

Se puede indicar una función de validación de los datos para todos los controles del formulario. También podemos importar `Validators` para obtener algunos validadores básicos como `Validators.required` para indicar un campo obligatorio, `Validators.email` que simplemente valida que un email sea válido y `Validators.pattern(regex)` que permite indicar una expresión regular que se debe cumplir.

También se pueden crear validadores personalizados simplemente creando un método que recibe un control de formulario `AbstractControl` por parámetro y retorna `ValidationErrors | null`.

```
validarDato: ValidatorFn = (control: AbstractControl) : ValidationErrors | null=>{
    if esValido(control.value) return null
    else return {error: 'inválido'}
  };
```

Un campo puede tener más de un validador, por ejemplo `Validators.email` solamente valida que el email sea válido pero no que se haya rellenado. Si necesitamos que sea válido y obligatorio combinariamos los validadores `Validators.required` y `Validators.email` de la siguiente forma:

```
new FormControl('',Validators.email) // solo una validación
new FormControl('',[Validators.required,Validators.email]) // multiples validaciones
```

### Librerias

También se pueden crear librerias. De este modo podemos crear componentes, servicios y otras clases y reutilizarlas en varios proyectos.

Creamos una libreria con `ng g lib` o `ng generate library` con el nombre lib-reservas. Se creará la libreria en `projects/lib-reservas` y se agregará en `projects` dentro de `angular.json`. En este caso solo voy a mover el componente de listar reservas y los servicios a esta libreria, debido a que son los que más podria necesitar si creo algun otro proyecto.

Ahora sería simplemente mover los archivos que se corresponden a los servicios de reserva, api y el componente a la carpeta de la librería. Por suerte ninguno depende de ninguna otra clase así que todos pueden salir del proyecto. Por ejemplo en el componente de reserva tenemos un @Output que emite un evento con datos de reserva. Lo estamos capturando en el `app.component.ts`, pero no dependemos de nada del componente en si, simplemente es el componente `app` que se encarga de utilizarlo.

Eso no significa que al moverlo todo vaya a seguir funcionando. Primero habra que actualizar los imports, aunque el VS Code ya puede hacerlo automáticamente. Luego también hay que revisar el `app.module.ts`. Tenemos importado el `HttpClientModule` que utilizaba el `ApiService` pero ya lo hemos sacado. Habría que moverlo a la librería, la cual no forma parte de un módulo, asi que lo creamos. Nos movemos con `cd projects/lib-reservas` a la librería y lo creamos con `ng g m reservas` (o `ng generate module reservas`). Lo situamos con el resto de componentes a nuestro gusto pero lo declaramos en `public-api.ts`, donde se indican los objetos exportados para importar desde el proyecto. Tambien sacar el componente `ListReserva` del `app.module` de la app y moverlo en el `reservas.module.ts` de la libreria. Revisar que este todo en su sitio, dentro del proyecto `web-reserva` solo deberíamos importar en el `app.module` el `ReservasModule`.

Si todo ha ido bien no deberia haber ningun error de imports y el servidor de la app que arrancamos con `npm run start` deberia seguir funcionando, aunque al mover algunos archivos seguramente necesite un reinicio para encontrarlos. La app web-reservas debería seguir funcionando y el listado debería mostrar los datos y permitir editar. De este modo si lo queremos usar en otro proyecto solo tendremos que importar `ReservasModule` para que funcione.

Por ejemplo voy a crear un segundo proyecto de aplicación `ng g app web-gestion-reservas` que sirva para obtener de la API todas las reservas y permitir gestionarlas completamente. Dentro del `web-gestion-reservas/src/app/app.module.ts` solamente necesito importar `ReservasModule`. Ahora en el `app.component.html` ya puedo utilizar el componente `<app-list-reservas>` y en el `app.component.ts` puedo inyectar el `ReservaService` en el constructor.

### Proyecto 2

Sin entrar en detalles en el proyecto `web-gestion-reservas` he añadido un formulario para editar cualquier campo pero en este caso utilizamos directivas *ngFor para generarlo. Esto puede ser útil en formularios muy grandes. Aquí aparece el `FormBuilder` que permite generar un formulario a partir de un objeto que hayamos generado. En resumen los cambios son:

 - Corregir método delete en `ApiService` de la libreria.
 - Corregir método `getReservas` en `ReservaService` de la libreria.
 - Método para eliminar en `ReservaService` en la libreria.
 - Componente `app-gestionar-reserva` que permite modificar los datos de la reserva seleccionada en el listado. En este caso uso un @ViewChild en el padre para obtener el componente hijo y asignarle el valor.
 - Directiva `PostTemplate` que de forma similar a un ngIf se evalua para manipular el template al que se le aplica, pero le asignamos una función personalizada para que haga lo que queramos.


```
<app-list-reserva (modificarReserva)="modificar($event)"></app-list-reserva>
```
```
  constructor(reservas:ReservaService) {
    reservas.getReservas()
  }

  modificar(datos:DatosReserva) {
    console.log(datos)
  }
```

Para arrancar este proyecto terminamos el anterior y lo arrancamos con `npm run start web-gestion-reservas`. Con solamente esto ya deberiamos ver el listado y al hacer click en editar se muestra en la consola los datos de la reserva.

## Development server

Run `ng serve` for a dev server. Navigate to `http://localhost:4200/`. The application will automatically reload if you change any of the source files.

## Code scaffolding

Run `ng generate component component-name` to generate a new component. You can also use `ng generate directive|pipe|service|class|guard|interface|enum|module`.

## Build

Run `ng build` to build the project. The build artifacts will be stored in the `dist/` directory.

## Running unit tests

Run `ng test` to execute the unit tests via [Karma](https://karma-runner.github.io).

## Running end-to-end tests

Run `ng e2e` to execute the end-to-end tests via a platform of your choice. To use this command, you need to first add a package that implements end-to-end testing capabilities.

## Further help

To get more help on the Angular CLI use `ng help` or go check out the [Angular CLI Overview and Command Reference](https://angular.io/cli) page.
