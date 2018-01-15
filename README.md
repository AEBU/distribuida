
Repaso Distribuida
-
En este post pondremos cuales son los pasos para usar las diferentes herramientas dadas en clases, con una descripción detallada de lo que significa cada configuración.

## Temario 1er post  `Uso de artemis y configuración`
* Qué es, como funciona y configuración de artemis
* Tipos de configuraciones de configurarlo y limitaciones
    * Creación del consumidor 
    * Creación del productor


## [Artemis][1]
-   Sistema de mensajería asíncrono, clúster  de alto rendimiento, multiprotocolo e integrable con:
    -   HTTP(S)
    -   IP MULTICAST
    -   SSL
    -   STOMP
    -   TCP
    -   UDP
    -   XMPP
-   Es MOM (Middleware oriented of messages), crea una capa de comunicación distribuida que aisla al desarrollador de la aplicación de los diferentes sistemas operativos e interfaces de red.
-   Tiene una aquitectura no bloqueante
-   Soporte del protocolo HornetQ Core para clientes Hornet
-   Compatibilidad con JMS 2.0 y 1.1
## Manual, important features
-   Estilos de mensajería
    -   Cola queue
        -   puede haber muchos consumidores en una cola, pero un mensaje en particular solo será consumido por un máximo de uno de ellos
    -   publicador/subscriptor
        -   Puede haber varios subscriptores para un tema, cada subscriptor recibe una copia de cada mensaje enviado al tema
-   Durabilidad
    -   duraderos(Pedidos no se pueden perder)
        -   Sobreviven a error del servidor o al reinicio.
    -   no DUraderos(actualización del precio del inventario)
        -   No sobreviven al error o reinicio del servidor
-   ¿Como Interactúan las aplicaciones cliente con los sistemas de mensajería para enviar y consumir mensajes?
    -   Mensajería
        -   APIs
        -   Protocolos de mensajería
    -   JMS
        -   Es una API, solo está disponible para clientes que ejecutan JAVA
    
    -   Protocolos que puede usar
        -   AMQP
            -   Es una especificación de mensajería interoperable, sus clientes están disponibles en muchos lenguajes de programación diferentes (AMQP1.0)
        -   STOMP
            -   Define un formato de conexión, cualquier cliente de Stomp, puede trabjar con cualquier sistema de mensajería que admita STOMP.
        -   OPENWIRE
        -   HornteQ (Para usar con clientes HornetQ)   
        -   Core (protocolo Artemis CORE)

# Usando el Servidor Artemis ActiveMQ

-   Debemos configurar una instancia de broker.xml
    -   ${ARTEMIS_HOME}/bin/
        -    Ejecuto  ./artemis create mydirect
            -   username
            -   password
            -   y [allow anonymous access]
-   Para iniciar
    -   ${ARTEMIS_HOME}/bin/mydirect/bin/
        -   Ejecuto ./artemis run
-   Para terminar el servidor
    -   ./artemis stop
-   Configuración de las colas
    -   autodele
    -   auto create
    -   direcciones
        -   anycast colas
        -   multicast pub/sub

-   Consideraciones: 
    -   Un servidor mantiene una asignación entre una dirección y un conjunto de colas. Cero o más colas pueden vincularse a una sola dirección. Cada cola puede vincularse con un filtro de mensaje opcional. Cuando se enruta un mensaje, se enruta al conjunto de colas vinculadas a la dirección del mensaje. Si alguna de las colas está vinculada con una expresión de filtro, el mensaje solo se enrutará al subconjunto de colas vinculadas que coincidan con esa expresión de filtro.
# Direccionamiento y colas

-   el modelo de direccionamiento comprende tres conceptos
    -   direccioens
    -   colas
    -   tipos de enrutamiento

---
# Configuración servidor


> Tenemos una aplicación war que se deployará en un contenedor web (tomcat), en este caso daremos a conocer los pasos para poder realizar estas acciones ya que esto debe cumplir el servidor.
-       Crear una base de datos ya sea con JPA o con JDBC
-       Exportar servicio por HttpInvoker
-       Consumir cola por HornetQ
-       Consumir cola por ArtemisMQ

1.- Para crear la configuracion de la base de datos:
> Para HttpInvoker
---
> Con JDBC
-   Creamos la clase que configura la base de datos ydentro la configuración de la base de datos, en este caso como estamos con Derby pues hacemos la configuración necesaria para esta. Podemos añadir script de base de datos a la que estamos creando, tomando siempre en cuenta la dirección del `classphat:` en adelante por ejemplo yo tengo mi archivo de configuración en:
    -   resources/db/schema.sql
    -   resources/db/test-data.sql
    -   Quedando como resultado a `.addScript( "classpath:sql/schema.sql" )`
- Siempre creamos atributos para cada tabla en este caso podemos dar a conocer algunos que nos ayudarán cuando tengamos que definir uno en particular.
    -   Para generar un ID como identity
        -    ID INT NOT NULL GENERATED ALWAYS AS IDENTITY
    -   Para formar un primary key
        -   PRIMARY KEY (INSTRUMENT_ID)
    -   Para usar un constraint 
        -   CONSTRAINT FK_ALBUM_SINGER FOREIGN KEY (SINGER_ID) REFERENCES SINGER (ID)
    -   Para Date
        -   BIRTH_DATE DATE
-   Como es una aplicacion web necesitamos @EnableWebMvc y la configuramos sin ninguna logica dentro solo necesitamos nuetra anotaciónm ya que necesitamos exponer un servicio http.
-   Ahora tenemos un apartado interesante, donde debemos configurar algunas cosas cuando se trata del rootappContext o el web appContext
    -   Creamos la clase que herede de AbstractAnnotationConfigDispatcherServletInitializer la cual me permite evitar estar configurando con un archivo web.xml. Esta clase importa la configuración para:
        -   rootContext
            -   Acceso a datos, transacciones y servicios de configuración
            -   crea el contexto de la aplicación raiz en función de él.
        -   webContext
            -   Controllers, ViewResolver, HandlerMapping, etc
            -   crea el contexto de la aplicación web, se crea utilizando la clase WebConfig y la clase de confiraucioón que define la configuración para el servicio de invocador HTTP.   
        -   En este [link][2] tenemos una definición detallada sobre la diferencia y herencia de getrootAplication, y getwebAplication los cuales difieren en que se crea un solo rootAplication, pero se puede tener varios dispatcherServlets para una aplicación web(Dispatcher servlet es el que mapea las solicitudes a cada vista especificada).

- Cuando usar http Invoker
    -   Si ambas aplicaciones necesitan comunciarse (Cliente Servidor), y estan basadas en Spring, HttpInvoker provee una simple y eficiente manera de invocar los servicios expuestos por otras aplicaciones.

- Configuración de HttpInvoker
    -   Usamos httpInvokerServiceExporter, se define con la clase HttpInvokerServiceExporter, que sirve para exportar cualquier bean de Spring como servicio a través de HttpInvoker dentro tenemos dos propiedades:
        -   Prop. del servicio, que indica el bean que proporciona el servicio
        -   Prop. interfaz, se expone el tipo de interfaz 
Conseguimos que nuestra capa de servicio está lista para ser expuesta y utilizada por clientes remotos.

-   No nos debemos olvidar que las entidades deben ser serializables y poner los log4j para ver como funciona o si esta dando nuestros servicios necesarios






## Aplicacion Cliente

>En este apartado pues expondremos nuestra aplicación para que el cliente la consuma y pueda realizar las peticiones al servicio expuesto.

-   Creamos las dependencias necesarias dentro del archivo .gradle.
-   Lo que trateremos de hacer es que tendremos que consumir el servicio de parte del servidor para poder realizar una operacion CRUD para ello pues haremos lo siguiente:
    -   Tenemos qeu tener los mismos paquetes en las aplicaciones tanto servidor como cliente.
    -   Debemos tener las entities que se formar'an para poder enviar y recibir nuestros parametros necesarios 
    -   Debemos tener las interfaces que se han resuelto en nuestro servidor.
> Como nota los paquetes quedar'ian de la siguiente forma:
-   dto
    -   entities
-   Interfaces
    -   interfaces

>Para configurar el cliente que pueda recibir los servicios expuestos y consumirlos tenemos que:

-   Crear la clase de configuración que se necesita para consumir, la cual declara:
    -   Bean HttpInvokerProxyFactoryBean 
        -   Dos propiedades dentro de este:
            -   serviceUrl que especifica la ubicación del servicio remoto
            -   la interfaz del servicio: que es la misma que interfaz que está dentro del servidor.
    -   Web Initializer, es la clase que se iniciará con las configuraciones que nosotros le demos para poder configurarla
        -   para omitir el web.xml debemos usar el AnnotationConfigWebApplicationContext, que lo usaremos para registrar nuestros archivos de configuraci'on y para omitir el ContextLoaderListener del web, pues al contexto del servlet le añadimos el listener necesario sobre el contexto declarado.
> Para el manejo de los beans jsf pues usamos las anotacioones necesarias que son @ManagedBean(name=...), y @SessionScope y para insertar un servicio definido desde Spring debemos traerlo desde la anotación @PostConstruct, 
        
-   ApplicationContext context =
                FacesContextUtils.getWebApplicationContext(FacesContext.getCurrentInstance());
-   con esto podemos iniciar el contexto dado
    -    servicio = context.getBean( "singerService", SingerService.class );
    -   singers = servicio.listar( );

    public String listar( ) {

        singers = servicio.listar( );

        return "";
    }
> Debemos tener en cuenta que este beanAction es elq ue se encargar'a de la lógica que se ha definido para el CRUD necesario.


    



----
# Luego veremos

-   Como configurar cliente - servidor y limitaciones con jsf, jpa o jdbc
- Jpa es la especificación  uqe dice como mapear los objetos a la base de datos  y hIbernate es la implementación lo que permite que ese código ejecute y suceda algo cuando lo corremos.
-   Verficiar como funciona con las 3 formas de usar JMS
- cliente recibe y publica en una cola.
- Probar con una entidad que se manda como objeto relacionado con otro objeto, es decir una entidad compuesta.
- Header en los mensajes de activeMQ
- consumir desde una cola en un servvidor y reenviar mesnajes a otra cola en un servidor diferente
-   pilas en la parte del problema del directorio con webapp afuera era hab'ia un bean con jsp.
-   Verificar los datos cuando es root App y web App context aqu'i hab'ia algo para configurar qeu decía si lo que hacíamos era necesario como root o como web.
-   Como funciona httpInvoker
-   Conceptos
    -   QUe es JMS
    -   HttpInvoker con búsquedas lookup y JNDI



[1]:https://activemq.apache.org/artemis/docs/latest/
[2]:https://stackoverflow.com/questions/35258758/getservletconfigclasses-vs-getrootconfigclasses-when-extending-abstractannot