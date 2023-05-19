#include <WiFi.h>        // Incluye la librería de WiFi
#include <HTTPClient.h>  // Incluye la librería de HTTPClient

const char* ssid = "HUAWEInovaY70";                             // Nombre de la red WiFi
const char* password = "16alejandro#";                       // Contraseña de la red WiFi
const char* botToken = "6109451610:AAFjO_tiBKT_FDEqAnuBgS2nsOa2JBGf7eQ";  // Token del bot de Telegram
const char* chatID = "6093286203";                        // ID del chat de Telegram

int hallSensorPin = 27;  // Define el pin en el que está conectado el sensor de infrarrojos
boolean sendNotifications = true; //esta variable se utiliza para habilitar o desabilitar el envio de notificaciones 

void setup() {
  Serial.begin(9600);           //inicializamos la comunicacion con el monitor serie
  WiFi.begin(ssid, password);   // establecemos la conexion wifi 
  
  while (WiFi.status() != WL_CONNECTED) {       //esblecemos un bucle while que se ejectura mientras que se establezca la coexion a wifi
    delay(1000);  //se repite la accion cada 1 segundo
    Serial.println("conectando a wifi...");     //este es el mensaje que se muestra mientras el bucle while se ejecuta
  }

  Serial.println("conectado a wifi");           //cuando se halla establecido la red wifi se muestra este mensaje 
}

void loop() {
  int hallSensorValue = digitalRead(hallSensorPin);     //se lee el estado del pin de manera digital 
  
  if (hallSensorValue == HIGH) {       //si hallsensor tiene un valor igual a LOW se indica  que no se ha detectado la presencia de un 
                                        //campo magnetico, por ende envia un mensaje. 
  
    Serial.println("movimiento detectado");   //este es el mensaje que se envia al no detectar campo
    sendTelegramMessage();      //envia un mensaje de notificacion a traves de telegram
    
  } else {                //si el sensor  tiene un valor diferente de LOW se ejecuta este bloque de codigo 
    sendNotifications = false;
    Serial.println("Movimiento no detectado");      //mensaje que se envia al detectar campo magnetico 
  }
  
 
  
  delay(1000);// espera un segundo para volver a repetir la accion
}

void sendTelegramMessage() { 
  HTTPClient http; //se crea una instancia de la clase HTTPclient llamada http esta instancia si utilizara para enviar solicitudes http
  
  String url = "https://api.telegram.org/bot"; // se crea una variable url de tipo String y se  inicializa con la base de la url de la
                                               //API de telegram
  //aqui se estructura la variable url para que se envien los mensajes correctamete
  
  url += botToken;            //se llama al token del bot de telegram
  url += "/sendMessage?chat_id="; 
  url += chatID;             //se llama el chat id del bot de telegram 
  url += "&text=";
  url += "movimiento detectado!";        //este es el mensaje que se envia al no detectar campo magnetico
  
  Serial.println("URL:"+ url);      //se conectan las partes de la url el token del bot, la ruta para enviar el mensaje el chat id y el                                       //texto del mensaj. 
  
  http.begin(url);   //se imprime en el monitor serie la url completa que se utiliza pra enviar la solicitud HTTP
  int httpCode = http.GET(); //se realiza una solicitud GET a la url utilizando http.Get y se almacena el codigo de respuesta HTTP en la
                              //variable httpcode
  
  if (httpCode == HTTP_CODE_OK) {// se verifica si el codigo de respuesta HTTP almacenado en httpcode es igual a HTTP CODE OK indicando
                                  //que la solicitud se realizo correctamente 
    Serial.println("Notification sent successfully");// si el codigo de respuesta es igual a HTTP CODE OK se muestra este mesaje
  } else {            // si el codigo de respuesta no es HTTP CODE OK  se muestra en el monitor serie el mensaje de la siguiente linea 
    Serial.println("Error sending notification");
  }
  
  http.end(); //se finaliza la solicitud HTTP mediante  http.end, esto libera los recursos utilizados por la solicitud y cierra la conexion.
}
