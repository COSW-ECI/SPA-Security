###Escuela Colombiana de Ingeniería
###Construcción de Software - COSW

####Laboratorio - SPA y Seguridad back-end.


1. Recupere el proyecto realizado en el ejericicio anterior (la aplicación de 'tareas pendientes').
2. V 
2. 
Continuar ejemplo lista TODO

1. Crear recurso REST para los TODO
2. Crear STUB de servicios de manejo de TODOs (+logs)
2. Implementar esquema de seguridad para evitar acceso directo. Pruena con CURL
3. Agregar vista de login, en el controlador: envío de petición a /user con encabezados HTTP
4. Agregar mecanismo de intercambio en Angular Route
5. Implementar controlador de Login: petición al recurso $USER, enviando encabezado HTTPBASIC con usuario/contraseña.
6. Al módulo de servicios, agregar una fábrica para los 
7. recursos 'todos'

6. Condicionar la visibilidad de los elementos (ng-show)

Probar

1. Mecanismo de protección de recursos por rol (consulta de TODOs, registro de TODOs).
2. Configurar usuarios para tener dos con difernetes roles.

Probar

1. Implementar Logout:
    * controlador en el módulo raíz.    
    * Agregar parámetro a configuración
    * método que envía petición a /logout
    * En la barra: <a href="" ng-click="m1()" >
    

Probar





CORS






<dependencies>
    ...
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
    ...
</dependencies>



Configurar WebSecurity


Inyección de dependencias


Angular resources
	* depedencias

	
	
config(['$routeProvider','$httpProvider', function($routeProvider, $httpProvider) {
  $routeProvider.otherwise({redirectTo: '/login'});
  $httpProvider.defaults.headers.common['X-Requested-With'] = 'XMLHttpRequest';
}]);



    @EnableGlobalMethodSecurity(prePostEnabled = true)