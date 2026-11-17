# Paket Soal C - Pemrograman Mobile

**Nama:** Rischa Sulistya Agustin
**NIM:** 241080200005
**Kelas:** Teknik Informatika 4B1

---

## 1. Analisis Alur Navigation Compose (20 poin)

**a) Halaman yang pertama kali dibuka**

Halaman `HomeScreen`, karena di `NavHost` ada `startDestination = "home"`. Jadi pas app jalan, route `"home"` yang ditampilkan duluan.

**b) Route yang dipanggil saat tombol detail ditekan**

Route `"detail/42"`. Tombol detail ada di `HomeScreen` lewat callback `onDetail`, dan isinya:

```kotlin
navController.navigate("detail/42")
```

**c) Dari mana nilai `id` diperoleh pada halaman detail**

Dari argument yang ikut di route. Route-nya `"detail/{id}"`, jadi `{id}` bakal keisi `42` (dari `detail/42`). Nilainya diambil lewat:

```kotlin
val id = entry.arguments?.getString("id") ?: "-"
```

`entry` itu `NavBackStackEntry`, dan `arguments`-nya nyimpen nilai argument dari route.

**d) Nilai pengganti jika `id` tidak ditemukan**

String `"-"`. Ini dari operator Elvis `?:`. Kalau `getString("id")` hasilnya `null` (atau `arguments` nya null), maka dipakai nilai default `"-"` supaya `DetailScreen` nggak dapat nilai null.

**e) Fungsi `NavHost`**

`NavHost` itu wadah (container) yang nampilin halaman sesuai route yang lagi aktif. Dia nentuin halaman awal (`startDestination`) dan daftar halaman yang bisa dituju (`composable("home")`, `composable("detail/{id}")`). Perpindahan antar halaman dikendalikan sama `navController`.

---

## 2. Perbaikan Penerapan State Hoisting (20 poin)

**a) Kode hasil perbaikan**

State `teks` dipindah ke `ParentScreen`, lalu `InputNama` cuma nerima `value` dan callback `onValueChange`.

```kotlin
@Composable
fun ParentScreen() {
    var teks by remember { mutableStateOf("") }

    InputNama(
        value = teks,
        onValueChange = { teks = it } // callback: parent yang update state
    )
}

@Composable
fun InputNama(
    value: String,
    onValueChange: (String) -> Unit
) {
    TextField(
        value = value,
        onValueChange = onValueChange,
        label = { Text("Nama") }
    )
}
```

**b) Kenapa disebut state hoisting**

Disebut *hoisting* (diangkat) karena state yang tadinya ada di dalam komponen anak (`InputNama`) diangkat naik ke parent (`ParentScreen`). Dengan begitu `InputNama` jadi **stateless**: dia cuma nampilin `value` yang dikasih dan ngasih tau parent lewat callback kalau ada perubahan. Keuntungannya, komponen lebih gampang dipakai ulang dan dites, dan state-nya punya satu sumber yang jelas (single source of truth).

---

## 3. LazyColumn dan Card untuk Daftar Tugas (20 poin)

**a) Kode Composable**

```kotlin
data class Tugas(val judul: String, val tanggal: String)

@Composable
fun DaftarTugas() {
    val listTugas = listOf(
        Tugas("Tugas Pemrograman Mobile", "20 November 2026"),
        Tugas("Tugas Audit Sistem", "25 November 2026")
    )

    LazyColumn(
        modifier = Modifier.fillMaxSize(),
        contentPadding = PaddingValues(16.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        items(listTugas) { tugas ->
            Card(modifier = Modifier.fillMaxWidth()) {
                Row(
                    modifier = Modifier
                        .fillMaxWidth()
                        .padding(16.dp),
                    verticalAlignment = Alignment.CenterVertically,
                    horizontalArrangement = Arrangement.SpaceBetween
                ) {
                    Column {
                        Text(text = tugas.judul)
                        Text(text = "Dikumpulkan: ${tugas.tanggal}")
                    }
                    Button(onClick = { /* aksi detail */ }) {
                        Text("Detail")
                    }
                }
            }
        }
    }
}
```

**b) Fungsi LazyColumn pada kasus ini**

`LazyColumn` dipakai buat nampilin daftar tugas yang bisa di-scroll secara vertikal. Bedanya sama `Column` biasa, `LazyColumn` cuma nge-compose item yang kelihatan di layar (lazy). Jadi kalau tugasnya banyak, performanya tetap ringan dan nggak boros memori. Dengan `items(listTugas)`, tiap data otomatis jadi satu `Card` tanpa harus nulis satu-satu.

---

## 4. Rancangan UI Menu Utama jadi Source Code (25 poin)

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun MenuUtamaScreen() {
    val menuList = listOf("Profil", "Jadwal", "Nilai", "Tagihan")

    Scaffold(
        topBar = {
            CenterAlignedTopAppBar(title = { Text("Menu Akademik") })
        }
    ) { innerPadding ->
        LazyVerticalGrid(
            columns = GridCells.Fixed(2),
            modifier = Modifier
                .fillMaxSize()
                .padding(innerPadding),
            contentPadding = PaddingValues(16.dp),
            horizontalArrangement = Arrangement.spacedBy(12.dp),
            verticalArrangement = Arrangement.spacedBy(12.dp)
        ) {
            items(menuList) { menu ->
                MenuCard(
                    nama = menu,
                    onClick = {
                        // callback sederhana tiap kartu
                        when (menu) {
                            "Profil" -> { /* ke halaman Profil */ }
                            "Jadwal" -> { /* ke halaman Jadwal */ }
                            "Nilai" -> { /* ke halaman Nilai */ }
                            "Tagihan" -> { /* ke halaman Tagihan */ }
                        }
                    }
                )
            }
        }
    }
}

@Composable
fun MenuCard(nama: String, onClick: () -> Unit) {
    Card(
        onClick = onClick,
        modifier = Modifier
            .fillMaxWidth()
            .height(100.dp)
    ) {
        Box(
            modifier = Modifier.fillMaxSize(),
            contentAlignment = Alignment.Center
        ) {
            Text(text = nama)
        }
    }
}
```

Penjelasan singkat:
- `LazyVerticalGrid` dengan `GridCells.Fixed(2)` bikin menu tampil 2 kolom sesuai wireframe.
- Tiap menu dibungkus `Card` yang bisa ditekan (`onClick`).
- Callback `onClick` dibikin sederhana dulu, belum perlu halaman tujuan.

---

## 5. Source Code jadi Pseudocode (15 poin)

**Pseudocode**

```
BUAT daftar jadwalHariIni KOSONG

UNTUK SETIAP item DALAM daftar jadwal:
    JIKA item.hari SAMA DENGAN "Senin":
        teks = item.jam + " - " + item.mataKuliah
        TAMBAHKAN teks KE jadwalHariIni

HASIL = jadwalHariIni
```

**a) Data yang dipilih oleh `filter`**

Data jadwal yang `hari`-nya `"Senin"`. Jadwal hari lain dibuang.

**b) Informasi yang diambil oleh `map`**

Properti `jam` dan `mataKuliah` dari tiap jadwal yang lolos filter, lalu digabung jadi satu string berformat `"jam - mataKuliah"`.

**c) Bagaimana urutan data dipertahankan**

`filter` dan `map` memproses elemen satu per satu dari awal sampai akhir dan nggak mengubah urutannya. Jadi urutan hasil akhir sama kayak urutan data asli di list `jadwal`. Gaya penulisannya juga deklaratif, jadi kita cukup nulis *apa* yang diinginkan (filter lalu map), bukan *bagaimana* looping-nya.
