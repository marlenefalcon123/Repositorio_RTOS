#if CONFIG_FREERTOS_UNICORE
  static const BaseType_t app_cpu = 0;
#else
  static const BaseType_t app_cpu = 1;
#endif

// Pines LEDs Semáforo 1
const int ROJO_01 = 19;
const int VERDE_01 = 5;

// Pines LEDs Semáforo 2
const int ROJO_02 = 15;
const int VERDE_02 = 4;

// Pines de Switches
const int SW_01 = 0;  // esperando en semáforo 1
const int SW_02 = 2;  // esperando en semáforo 2
const int SW_03 = 18; // 1 salió
const int SW_04 = 21; // 2 salió

// sección critica con MuteX
static SemaphoreHandle_t xMutexCarretera;

// declarar tareas
void tareaSemaforo1(void *parameters);
void tareaSemaforo2(void *parameters);

void setup() {
  Serial.begin(115200);

  // Config leds
  pinMode(ROJO_01, OUTPUT);
  pinMode(VERDE_01, OUTPUT);
  pinMode(ROJO_02, OUTPUT);
  pinMode(VERDE_02, OUTPUT);

  // Config Switches
  pinMode(SW_01, INPUT_PULLUP);
  pinMode(SW_02, INPUT_PULLUP);
  pinMode(SW_03, INPUT_PULLUP);
  pinMode(SW_04, INPUT_PULLUP);

  // Estados iniciales leds
  digitalWrite(ROJO_01, HIGH);
  digitalWrite(VERDE_01, LOW);
  digitalWrite(ROJO_02, HIGH);
  digitalWrite(VERDE_02, LOW);

  // Crear el mutex
  xMutexCarretera = xSemaphoreCreateMutex();

  // Crear tareas
  xTaskCreatePinnedToCore(tareaSemaforo1, "Semaforo1", 2048, NULL, 1, NULL, app_cpu);
  xTaskCreatePinnedToCore(tareaSemaforo2, "Semaforo2", 2048, NULL, 1, NULL, app_cpu);
}

void loop() {
  // No se usa en FreeRTOS
}

// Tarea del semaforo 1
void tareaSemaforo1(void *parameters) {
  while (1) {
    // Detectar coche en semáforo 1
    if (digitalRead(SW_01) == LOW) {
      // detecta coche 1
      
      // Intentar acceder a la carretera -sección crítica
      if (xSemaphoreTake(xMutexCarretera, portMAX_DELAY) == pdTRUE) {
        //coche 1 en carretera
        digitalWrite(ROJO_01, LOW);
        digitalWrite(VERDE_01, HIGH);

        // Mientras no se haya detectado que salió 
        while (digitalRead(SW_03) == HIGH) {
          vTaskDelay(100 / portTICK_PERIOD_MS);
        }

        // Coche salió
        digitalWrite(VERDE_01, LOW);
        digitalWrite(ROJO_01, HIGH);

        // Liberar carretera
        xSemaphoreGive(xMutexCarretera);
      }
    }

    vTaskDelay(100 / portTICK_PERIOD_MS);
  }
}

// Sem 2

void tareaSemaforo2(void *parameters) {
  while (1) {
    // Detectar coche en semáforo 2
    if (digitalRead(SW_02) == LOW) {

      // Intentar acceder a la carretera coche 2 -sección crítica
      if (xSemaphoreTake(xMutexCarretera, portMAX_DELAY) == pdTRUE) {

        digitalWrite(ROJO_02, LOW);
        digitalWrite(VERDE_02, HIGH);

        // Mientras no se haya detectado que salió esperar
        while (digitalRead(SW_04) == HIGH) {
          vTaskDelay(100 / portTICK_PERIOD_MS);
        }

        // Coche 2 salió
        digitalWrite(VERDE_02, LOW);
        digitalWrite(ROJO_02, HIGH);

        // Liberar carretera
        xSemaphoreGive(xMutexCarretera);
      }
    }

    vTaskDelay(100 / portTICK_PERIOD_MS);
  }
}
