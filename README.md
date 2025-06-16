# trampolin-distance-oben

const int trigPin = 9;
const int echoPin = 10;

const int NO_PERSON_THRESHOLD = 200; // Distance >= this value (cm) is considered no person detected
const int MIN_HEIGHT_CM = 140;       // Minimum valid height for base height registration

int baseHeight = 0;
bool personPresent = false;
bool baseCaptured = false;
int lastLevel = -1;

void setup() {
  Serial.begin(115200);
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
}

long getDistanceCM() {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);

  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  long duration = pulseIn(echoPin, HIGH);
  long distanceCM = duration * 0.034 / 2;
  return distanceCM;
}

void loop() {
  long distance = getDistanceCM();

  if (distance >= NO_PERSON_THRESHOLD) {
    // No person detected
    if (personPresent) {
      personPresent = false;
      baseCaptured = false;
      lastLevel = -1;
      Serial.println("No person detected.");
    }
  } else {
    // Person detected
    if (!personPresent) {
      personPresent = true;
      baseCaptured = false;
      Serial.println("Person detected.");
    }

    if (!baseCaptured) {
      if (distance <= MIN_HEIGHT_CM) {
        baseHeight = distance;
        baseCaptured = true;
        Serial.print("Base height set: ");
        Serial.print(baseHeight);
        Serial.println(" cm");
      } else {
        Serial.println("Person too short to register base height.");
        delay(50);
        return;
      }
    }

    int delta = baseHeight - distance;
    int level = 0;

    if (delta >= 10 && delta < 20) {
      level = 1;
    } else if (delta >= 20 && delta < 30) {
      level = 2;
    } else if (delta >= 30) {
      level = 3;
    } else {
      level = 0;
    }

    if (level != lastLevel) {
      Serial.print("Level: ");
      Serial.println(level);
      lastLevel = level;
    }
  }

  delay(50);
}
