# LineBot-V1

**LineBot-V1** adalah robot line follower sederhana yang dibangun menggunakan Wemos D1 Mini. Robot ini mengikuti garis menggunakan sensor TCRT5000 dan dapat menghindari halangan menggunakan sensor ultrasonik HC-SR04. Setelah mendeteksi dan menghindari halangan, robot akan kembali ke jalur garis dan melanjutkan perjalanannya.

## Fitur

- Mengikuti garis menggunakan sensor TCRT5000 IR
- Deteksi dan penghindaran halangan menggunakan sensor ultrasonik HC-SR04
- Menggunakan driver motor L298N untuk mengontrol dua motor DC
- Secara otomatis kembali ke jalur setelah menghindari halangan

## Komponen

1. **Wemos D1 Mini**: Pengendali utama
2. **TCRT5000**: Sensor garis inframerah (2 unit)
3. **HC-SR04**: Sensor ultrasonik untuk deteksi halangan
4. **L298N Motor Driver**: Untuk mengontrol motor kiri dan kanan
5. **Motor DC**: Dua motor untuk menggerakkan robot
6. **Sumber Daya**: 5V untuk motor dan sensor

## Konfigurasi Pin

### **Sensor Garis TCRT5000**:
| Sensor | Pin Wemos |
|--------|-----------|
| Kiri   | D1        |
| Kanan  | D2        |

### **Sensor Ultrasonik HC-SR04**:
| Pin   | Pin Wemos |
|-------|-----------|
| Trig  | D3        |
| Echo  | D4        |

### **Driver Motor L298N**:
| Pin Motor | Pin Wemos |
|-----------|-----------|
| IN1 (Motor Kanan) | D5 |
| IN2 (Motor Kanan) | D6 |
| IN3 (Motor Kiri)  | D7 |
| IN4 (Motor Kiri)  | D8 |
| VCC      | 5V |
| GND      | GND |

## Cara Penggunaan

1. **Persiapan**:
   - Sambungkan semua komponen sesuai dengan **Konfigurasi Pin** yang tertera di atas.
   - Pastikan sumber daya terhubung dengan benar ke Wemos D1 Mini, sensor, dan motor driver.

2. **Unggah Kode**:
   - Buka Arduino IDE dan pastikan kamu sudah menginstal board Wemos D1 Mini.
   - Salin kode yang tertera di bagian **Kode** ke dalam Arduino IDE.
   - Pilih port dan board yang sesuai, lalu unggah kode ke Wemos D1 Mini.

3. **Pengujian**:
   - Tempatkan robot pada jalur garis yang telah disiapkan.
   - Robot akan mulai mengikuti garis secara otomatis.
   - Jika robot mendeteksi halangan di depan menggunakan sensor ultrasonik, robot akan berhenti, menghindari halangan, dan kemudian kembali ke jalur garis.

## Penjelasan Kode

Perilaku robot dapat dibagi menjadi dua bagian utama:
1. **Mengikuti Garis**: 
   - Dua sensor TCRT5000 mendeteksi keberadaan garis hitam dan menyesuaikan kecepatan serta arah motor sesuai dengan input sensor.
   - Jika kedua sensor mendeteksi garis (LOW), robot akan bergerak maju. Jika hanya sensor kiri yang mendeteksi garis, robot akan berbelok kiri, dan sebaliknya.

2. **Penghindaran Halangan**:
   - Sensor ultrasonik terus-menerus memeriksa halangan di depan. Jika halangan terdeteksi dalam jarak tertentu (misalnya, 20 cm), robot akan berhenti dan melaksanakan rutinitas untuk menghindari halangan dengan belok kanan, maju sedikit, dan kemudian belok kiri untuk mencari kembali garis.

## Kode

Berikut adalah kode lengkap untuk LineBot-V1:

```cpp
// Pin setup for sensors, motors, and ultrasonic sensor
const int sensorKiri = D1;  // Pin sensor kiri (TCRT5000)
const int sensorKanan = D2; // Pin sensor kanan (TCRT5000)

const int motorKanan1 = D5; // IN1 for right motor (L298N)
const int motorKanan2 = D6; // IN2 for right motor (L298N)
const int motorKiri1 = D7;  // IN3 for left motor (L298N)
const int motorKiri2 = D8;  // IN4 for left motor (L298N)

// Pin untuk sensor ultrasonik
const int trigPin = D3; // Pin trig pada sensor ultrasonik
const int echoPin = D4; // Pin echo pada sensor ultrasonik

// Threshold jarak minimal untuk menghindari halangan (dalam cm)
const int jarakThreshold = 20;

void setup() {
  // Inisialisasi pin sensor garis
  pinMode(sensorKiri, INPUT);
  pinMode(sensorKanan, INPUT);

  // Inisialisasi pin motor sebagai output
  pinMode(motorKanan1, OUTPUT);
  pinMode(motorKanan2, OUTPUT);
  pinMode(motorKiri1, OUTPUT);
  pinMode(motorKiri2, OUTPUT);

  // Inisialisasi pin ultrasonik
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);

  // Memulai motor dengan kondisi off
  stopMotors();

  // Inisialisasi Serial Monitor
  Serial.begin(9600);
}

void loop() {
  // Membaca sensor garis
  int bacaKiri = digitalRead(sensorKiri);
  int bacaKanan = digitalRead(sensorKanan);

  // Mendapatkan jarak dari sensor ultrasonik
  int jarak = bacaJarak();

  // Jika ada halangan di depan
  if (jarak <= jarakThreshold) {
    stopMotors(); // Berhenti jika ada halangan
    menghindar(); // Panggil fungsi untuk menghindari halangan
  } else {
    // Perilaku line follower jika tidak ada halangan
    if (bacaKiri == LOW && bacaKanan == LOW) {
      maju();
    } else if (bacaKiri == LOW && bacaKanan == HIGH) {
      belokKiri();
    } else if (bacaKiri == HIGH && bacaKanan == LOW) {
      belokKanan();
    } else {
      stopMotors();
    }
  }

  delay(50); // Menambahkan sedikit delay agar pembacaan sensor lebih stabil
}

// Fungsi untuk menggerakkan motor maju
void maju() {
  digitalWrite(motorKanan1, HIGH);
  digitalWrite(motorKanan2, LOW);
  digitalWrite(motorKiri1, HIGH);
  digitalWrite(motorKiri2, LOW);
}

// Fungsi untuk belok kiri
void belokKiri() {
  digitalWrite(motorKanan1, HIGH);
  digitalWrite(motorKanan2, LOW);
  digitalWrite(motorKiri1, LOW);
  digitalWrite(motorKiri2, LOW);
}

// Fungsi untuk belok kanan
void belokKanan() {
  digitalWrite(motorKanan1, LOW);
  digitalWrite(motorKanan2, LOW);
  digitalWrite(motorKiri1, HIGH);
  digitalWrite(motorKiri2, LOW);
}

// Fungsi untuk menghentikan motor
void stopMotors() {
  digitalWrite(motorKanan1, LOW);
  digitalWrite(motorKanan2, LOW);
  digitalWrite(motorKiri1, LOW);
  digitalWrite(motorKiri2, LOW);
}

// Fungsi untuk membaca jarak dari sensor ultrasonik
int bacaJarak() {
  // Mengirimkan trigger pulse
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  // Membaca echo pulse
  long durasi = pulseIn(echoPin, HIGH);

  // Menghitung jarak dalam cm
  int jarak = durasi * 0.034 / 2;
  return jarak;
}

// Fungsi untuk menghindari halangan
void menghindar() {
  // Logika untuk menghindar
  belokKanan();      // Robot belok ke kanan
  delay(1000);       // Tunggu 1 detik
  maju();            // Setelah itu maju sedikit
  delay(1000);       // Tunggu 1 detik
  stopMotors();      // Berhenti sejenak
  delay(500);        // Tunggu setengah detik
  belokKiri();       // Kembali ke arah kiri setelah menghindar
  delay(1000);       // Belok ke kiri selama 1 detik
  
  // Mencari kembali garis setelah menghindar
  while (digitalRead(sensorKiri) == HIGH && digitalRead(sensorKanan) == HIGH) {
    maju();  // Maju terus sampai menemukan garis
  }
  stopMotors(); // Hentikan motor saat kembali ke garis
}
