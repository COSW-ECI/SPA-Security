###Escuela Colombiana de Ingeniería
###Construcción de Software - COSW

####Laboratorio - SPA y Seguridad back-end.


__Parte I.__

1. Recupere el proyecto realizado en el ejericicio anterior (la aplicación de 'tareas pendientes').

 
2. Cree un API REST que permita (1) registrar tareas pendientes, y (2) consultar las tareas pendientes registradas, es decir, debe soportar peticiones GET y POST (recuerde cumplir con los niveles de madurez de los API REST). Recuerde que puede hacerlo fácilmente en una nueva clase usando las siguientes anotaciones:

	```java
@RestController
@RequestMapping("/urldelrecurso")
@RequestMapping(value = "/{ruta}", method = RequestMethod.GET)

	```
3. Para implementar la funcionalidad de lado del servidor (consulta de tareas, registro de una), cree una interfaz que defina las operaciones requeridas. Agregue al controlador del API Rest un atributo que corresponda al tipo de la interfaz creada, con su respectiva anotación @Autowired.

4. Implemente un Stub para dicha interfaz (una implementación simulada), el cual mantenga los datos (los TODOs registrados) en memoria. Haga que dicha implementación sea la que se inyecte en el controlador del API REST poniendo en éste una anotación @Service.

5. Inicie la aplicación y verifique el funcionamiento del API con el comando curl. Ejemplo para hacer una petición POST:

	```java
curl -H "Content-Type: application/json" -X POST -d '{"username":"xyz","password":"xyz"}' http://localhost:3000/api/login
	```
6. Habilite el esquema de seguridad de SpringBoot, agregando 'security-starter' como dependencia (en el pom.xml):

	```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>                
	```
 
7. Repita el comando anterior, y verifique si aún es posible acceder directamente al API.
8. Agregue la siguiente configuración en la clase principal de la aplicación (la que tiene la anotación @SpringBootApplication):

	```java
@Configuration
    @EnableGlobalMethodSecurity(prePostEnabled = true)
    @Order(SecurityProperties.ACCESS_OVERRIDE_ORDER)
    protected static class SecurityConfiguration extends WebSecurityConfigurerAdapter {

        @Override
        protected void configure(AuthenticationManagerBuilder builder) throws Exception {
            builder.inMemoryAuthentication().withUser("user").password("password").roles("USER");
        }
        
        @Override
        protected void configure(HttpSecurity http) throws Exception {
            http
                    .httpBasic()
                    .and()
                    .authorizeRequests()
                    .antMatchers("/app/**","/logout","/login").permitAll()
                    .anyRequest().authenticated().and()
                    .logout().logoutSuccessUrl("/")
                    .and().csrf()
                    .csrfTokenRepository(csrfTokenRepository()).and()
                    .addFilterAfter(csrfHeaderFilter(), CsrfFilter.class);
        }

        private Filter csrfHeaderFilter() {
            return new OncePerRequestFilter() {
                @Override
                protected void doFilterInternal(HttpServletRequest request,
                        HttpServletResponse response, FilterChain filterChain)
                        throws ServletException, IOException {
                    CsrfToken csrf = (CsrfToken) request.getAttribute(CsrfToken.class
                            .getName());
                    if (csrf != null) {
                        Cookie cookie = WebUtils.getCookie(request, "XSRF-TOKEN");
                        String token = csrf.getToken();
                        if (cookie == null || token != null
                                && !token.equals(cookie.getValue())) {
                            cookie = new Cookie("XSRF-TOKEN", token);
                            cookie.setPath("/");
                            response.addCookie(cookie);
                        }
                    }
                    filterChain.doFilter(request, response);
                }
            };
        }

        private CsrfTokenRepository csrfTokenRepository() {
            HttpSessionCsrfTokenRepository repository = new HttpSessionCsrfTokenRepository();
            repository.setHeaderName("X-XSRF-TOKEN");
            return repository;
        }

    }
	```

9. En la SPA (AngularJS), siguiendo el esquema del laboratorio anterior, cree una nueva vista -con su respectivo controlador- para hacer login. Configure el enrutamiento de la nueva vista, y agruéguela a la barra de opciones de la aplicación.

10. Rectifique que en la configuración del módulo principal (el definido en app/app.js) se tenga configurado el parámetro $httpProvider.defaults.headers.common dela siguiente manera:


	```javascript
config(['$routeProvider','$httpProvider', function($routeProvider, $httpProvider) {
  $routeProvider.otherwise({redirectTo: '/login'});
  $httpProvider.defaults.headers.common['X-Requested-With'] = 'XMLHttpRequest';
}]);
	```
10. Para la vista de login, use el siguiente formulario:

	```html
<form role="form" ng-submit="login()">
	<div class="form-group">
		<label for="username">Username:</label> <input type="text"
			class="form-control" id="username" name="username" ng-model="credentials.username"/>
	</div>
	<div class="form-group">
		<label for="password">Password:</label> <input type="password"
			class="form-control" id="password" name="password" ng-model="credentials.password"/>
	</div>
	<button type="submit" class="btn btn-primary">Submit</button>
</form>
	```

10. Implemente el controlador de la vista de login. Haga que en dicho controlador se inyecte: $rootScope, $scope, $http y $location.

12. En el controlador, agregue las funciones authenticate() y login(), junto con la propiedad 'credentials' (la asociada a la vista mediante ng-model):

	```javascript
                var authenticate = function (credentials, callback) {

                    var headers = credentials ? {authorization: "Basic "
                                + btoa(credentials.username + ":" + credentials.password)
                    } : {};

                    $http.get('user', {headers: headers}).success(function (data) {
                        if (data.name) {
                            $rootScope.authenticated = true;
                        } else {
                            $rootScope.authenticated = false;
                        }
                        callback && callback();
                    }).error(function () {
                        $rootScope.authenticated = false;
                        callback && callback();
                    });

                };

                authenticate();
                $scope.credentials = {};
                $scope.login = function () {
                    authenticate($scope.credentials, function () {                        
                        if ($rootScope.authenticated) {
                            $location.path("/");
                            $scope.error = false;
                        } else {
                            $location.path("/login");
                            $scope.error = true;
                        }
                    });
                };
	```


14. Con el esquema planteado, la propiedad de $rootScope.authenticated siempre tendrá un valor de verdadero cuando el usuario esté autenticado, y falso en cualquier otro caso. Use la directiva 'ng-show' para que las vista que muestra las tareas pendientes sólo lo haga si el usuario esá autenticado:

	```html
<div  ng-show="authenticated">
...
</div>
<div  ng-show="!authenticated">
...
</div>

	```
15. Verifique el nuevo funcionamiento de la aplicación.

16. Para implementar el logout, agregue un controlador al módulo raíz (el definido en app/app.js), al cual se le inyecte $scope, $http, $rootScope, y $location. En dicho controlador, defina la función de logout():

	```javascript
$scope.logout = function () {
                    $http.post('/logout', {}).success(function () {
                        $rootScope.authenticated = false;
                        $location.path("/");
                    }).error(function (data) {
                        $rootScope.authenticated = false;
                    });
                };
	```
17. Al menú de la aplicación, agregue una nueva entrada de menú (href), la cual, en lugar de redirigir a una vista, ejecute el anterior método mediante la directiva ng-click.

18. Verifique el funcionamiento de la aplicación.
	
__Parte II.__

Ahora, va a mantener la información de las tareas pendientes en el servidor. Para hacer esto, va a usar las facilidades del módulo '$resources' de angular.

1. Agregue la dependencia "angular-resource": "~1.4.0" a bower, y actualice las dependencias de javascript (bower install).
2. Importe la anterior librería, agregándola en app/index.html:

	```
	<script src="bower_components/angular-resource/angular-resource.js"></script>
	```	
	
3. En el módulo donde definió los servicios en el ejercicio anterior, inyecte ngResource como dependencia:

	```
angular.module('services.servicesA', ['ngRoute', 'ngResource'])
	```	

4. En el anterior módulo agregue un nuevo servicio (factory), el cual retorne un objeto $resource que corresponda a un recurso REST. Por ejemplo, si se tuviera un recurso del tipo /items/{iditem}, la fábrica se definiría como:

	```
.factory('Items', function($resource) {
            return $resource('/items/:id'); 
        });
	```	
5. Modifique la funcionalidad de la aplicación para que en lugar de agregar y consultar a una variable del servicio, lo haga al servidor, a través del $resource que puede proveer el nuevo servicio. Revise la documentación de Angular $resource en [https://docs.angularjs.org/api/ngResource/service/$resource](https://docs.angularjs.org/api/ngResource/service/$resource).

6. Revise el funcionamiento de la aplicación.