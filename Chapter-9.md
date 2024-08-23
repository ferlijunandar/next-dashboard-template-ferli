# Bab 9

## Streaming

Pada bab sebelumnya, Anda belajar tentang berbagai metode rendering di Next.js. Kami juga membahas bagaimana pengambilan data yang lambat dapat memengaruhi performa aplikasi Anda. Mari kita lihat bagaimana Anda dapat meningkatkan pengalaman pengguna ketika ada permintaan data yang lambat.

### Dalam bab ini...

Berikut adalah topik yang akan kita bahas:

- Apa itu streaming dan kapan Anda menggunakannya.
- Cara mengimplementasikan streaming dengan loading.js dan Suspense.
- Apa itu loading skeletons.
- Apa itu route groups, dan kapan Anda menggunakannya.
- Dimana meletakkan batas Suspense di aplikasi Anda.

### Apa itu Streaming?

Streaming adalah teknik transfer data yang memungkinkan Anda memecah rute menjadi "chunks" yang lebih kecil dan mengalirkannya secara progresif dari server ke klien saat mereka siap.
![alt text](image-21.png)
Dengan streaming, Anda dapat mencegah permintaan data yang lambat memblokir seluruh halaman Anda. Ini memungkinkan pengguna untuk melihat dan berinteraksi dengan bagian halaman tanpa menunggu semua data dimuat sebelum UI apa pun dapat ditampilkan kepada pengguna.
![alt text](image-22.png)
Streaming bekerja dengan baik dengan model komponen React, karena setiap komponen dapat dianggap sebagai chunk.

Ada dua cara untuk mengimplementasikan streaming di Next.js:

- Pada tingkat halaman, dengan file loading.js.
- Untuk komponen tertentu, dengan `<Suspense>`.

### Streaming Seluruh Halaman dengan loading.js

Di folder /app/dashboard, buat file baru bernama loading.js:

```javascript
// /app/dashboard/loading.js
export default function Loading() {
  return <div>Loading...</div>;
}
```

Segarkan http://localhost:3000/dashboard, dan Anda sekarang harus melihat:
![alt text](image-23.png)

```
Dashboard page with 'Loading...' text
```

Beberapa hal terjadi di sini:

- loading.js adalah file khusus Next.js yang dibangun di atas Suspense, ini memungkinkan Anda membuat UI pengganti untuk ditampilkan sementara konten halaman dimuat.
- Karena `<SideNav>` bersifat statis, ini ditampilkan segera. Pengguna dapat berinteraksi dengan `<SideNav>` sementara konten dinamis dimuat.
- Pengguna tidak harus menunggu halaman selesai dimuat sebelum meninggalkan halaman (ini disebut navigasi yang dapat diinterupsi).

Selamat! Anda baru saja mengimplementasikan streaming. Tetapi kita dapat melakukan lebih banyak untuk meningkatkan pengalaman pengguna. Mari kita tampilkan loading skeleton sebagai pengganti teks Loading….

### Menambahkan Loading Skeletons

Loading skeleton adalah versi sederhana dari UI. Banyak situs web menggunakannya sebagai placeholder (atau fallback) untuk menunjukkan kepada pengguna bahwa konten sedang dimuat. Setiap UI yang Anda tambahkan dalam loading.js akan disematkan sebagai bagian dari file statis, dan dikirimkan terlebih dahulu. Kemudian, sisa konten dinamis akan dialirkan dari server ke klien.

Di dalam file loading.js Anda, impor komponen baru bernama `<DashboardSkeleton>`:

```javascript
// /app/dashboard/loading.js
import DashboardSkeleton from "@/app/ui/skeletons";

export default function Loading() {
  return <DashboardSkeleton />;
}
```

Kemudian, segarkan http://localhost:3000/dashboard, dan Anda sekarang harus melihat:
![alt text](image-24.png)

```
Dashboard page with loading skeletons
```

### Memperbaiki Bug Loading Skeleton dengan Route Groups

Saat ini, loading skeleton Anda akan berlaku untuk halaman invoices dan customers juga.

Karena loading.js berada pada tingkat yang lebih tinggi daripada /invoices/page.js dan /customers/page.js dalam sistem file, ini juga diterapkan pada halaman-halaman tersebut.

Kita bisa mengubah ini dengan Route Groups. Buat folder baru bernama /(overview) di dalam folder dashboard. Kemudian, pindahkan file loading.js dan page.js ke dalam folder tersebut:
![alt text](image-25.png)

```
Folder structure showing how to create a route group using parentheses
```

Sekarang, file loading.js hanya akan berlaku untuk halaman ikhtisar dashboard Anda.

Route groups memungkinkan Anda mengatur file ke dalam kelompok logis tanpa memengaruhi struktur jalur URL. Ketika Anda membuat folder baru menggunakan tanda kurung (), namanya tidak akan dimasukkan dalam jalur URL. Jadi /dashboard/(overview)/page.js menjadi /dashboard.

Di sini, Anda menggunakan route group untuk memastikan loading.js hanya berlaku untuk halaman ikhtisar dashboard Anda. Namun, Anda juga dapat menggunakan route groups untuk memisahkan aplikasi Anda menjadi bagian-bagian (misalnya rute (marketing) dan rute (shop)) atau berdasarkan tim untuk aplikasi yang lebih besar.

### Streaming Komponen

Sejauh ini, Anda melakukan streaming seluruh halaman. Tetapi Anda juga dapat lebih mendetail dan melakukan streaming komponen tertentu menggunakan React Suspense.

Suspense memungkinkan Anda menunda rendering bagian dari aplikasi Anda sampai beberapa kondisi terpenuhi (misalnya data dimuat). Anda dapat membungkus komponen dinamis Anda dalam Suspense. Kemudian, beri komponen pengganti untuk ditampilkan sementara komponen dinamis dimuat.

Jika Anda ingat permintaan data yang lambat, fetchRevenue(), inilah permintaan yang memperlambat seluruh halaman. Alih-alih memblokir seluruh halaman Anda, Anda dapat menggunakan Suspense untuk melakukan streaming hanya komponen ini dan segera menampilkan sisa UI halaman.

Untuk melakukannya, Anda perlu memindahkan pengambilan data ke komponen, mari kita perbarui kode untuk melihat bagaimana tampilannya:

Hapus semua instance fetchRevenue() dan datanya dari /dashboard/(overview)/page.js:

```javascript
// /app/dashboard/(overview)/page.js
import { Card } from '@/app/ui/dashboard/cards';
import RevenueChart from '@/app/ui/dashboard/revenue-chart';
import LatestInvoices from '@/app/ui/dashboard/latest-invoices';
import { lusitana } from '@/app/ui/fonts';
import { fetchLatestInvoices, fetchCardData } from '@/app/lib/data'; // hapus fetchRevenue

export default async function Page() {
  const revenue = await fetchRevenue() // hapus baris ini
  const latestInvoices = await fetchLatestInvoices();
  const {
    numberOfInvoices,
    numberOfCustomers,
    totalPaidInvoices,
    totalPendingInvoices,
  } = await fetchCardData();

  return (
    // ...
  );
}
```

Kemudian, impor `<Suspense>` dari React, dan bungkus di sekitar `<RevenueChart />`. Anda dapat memberinya komponen pengganti yang disebut `<RevenueChartSkeleton>`.

```javascript
// /app/dashboard/(overview)/page.js
import { Card } from "@/app/ui/dashboard/cards";
import RevenueChart from "@/app/ui/dashboard/revenue-chart";
import LatestInvoices from "@/app/ui/dashboard/latest-invoices";
import { lusitana } from "@/app/ui/fonts";
import { fetchLatestInvoices, fetchCardData } from "@/app/lib/data";
import { Suspense } from "react";
import { RevenueChartSkeleton } from "@/app/ui/skeletons";

export default async function Page() {
  const latestInvoices = await fetchLatestInvoices();
  const {
    numberOfInvoices,
    numberOfCustomers,
    totalPaidInvoices,
    totalPendingInvoices,
  } = await fetchCardData();

  return (
    <main>
      <h1 className={`${lusitana.className} mb-4 text-xl md:text-2xl`}>
        Dashboard
      </h1>
      <div className="grid gap-6 sm:grid-cols-2 lg:grid-cols-4">
        <Card title="Collected" value={totalPaidInvoices} type="collected" />
        <Card title="Pending" value={totalPendingInvoices} type="pending" />
        <Card title="Total Invoices" value={numberOfInvoices} type="invoices" />
        <Card
          title="Total Customers"
          value={numberOfCustomers}
          type="customers"
        />
      </div>
      <div className="mt-6 grid grid-cols-1 gap-6 md:grid-cols-4 lg:grid-cols-8">
        <Suspense fallback={<RevenueChartSkeleton />}>
          <RevenueChart />
        </Suspense>
        <LatestInvoices latestInvoices={latestInvoices} />
      </div>
    </main>
  );
}
```

Terakhir, perbarui komponen `<RevenueChart>` untuk mengambil datanya sendiri dan hapus prop yang diteruskan kepadanya:

```javascript
// /app/ui/dashboard/revenue-chart.js
import { generateYAxis } from '@/app/lib/utils';
import { CalendarIcon } from '@heroicons/react/24/outline';
import { lusitana } from '@/app/ui/fonts';
import { fetchRevenue } from '@/app/lib/data';

// ...

export default async function RevenueChart() { // Jadikan komponen async, hapus props
  const revenue = await fetchRevenue(); // Ambil data di dalam komponen

  const chartHeight = 350;
  const { yAxisLabels, topLabel } = generateYAxis(revenue);

  if (!revenue || revenue.length === 0) {
    return <p className="mt-4 text-gray-400">No data available.</p>;
  }

  return (
    // ...
  );
}
```

Sekarang segarkan halaman, Anda seharusnya melihat informasi dashboard hampir seketika, sementara kerangka fallback ditampilkan untuk `<RevenueChart>`:
![alt text](image-26.png)

```
Dashboard page with revenue chart skeleton and loaded Card and Latest Invoices components
```

### Latihan: Streaming `<LatestInvoices>`

Sekarang giliran Anda! Latih apa yang baru saja Anda pelajari dengan melakukan streaming komponen `<LatestInvoices>`.

Pindahkan `fetchLatestInvoices()` dari halaman ke komponen `<LatestInvoices>`. Bungkus komponen dalam batas `<Suspense>` dengan fallback yang disebut `<LatestInvoicesSkeleton>`.

Setelah

Anda siap, perluas toggle untuk melihat kode solusinya:

### Mengelompokkan Komponen

Bagus! Anda hampir selesai, sekarang Anda perlu membungkus komponen `<Card>` dalam Suspense. Anda dapat mengambil data untuk setiap kartu individu, tetapi ini bisa menyebabkan efek popping saat kartu dimuat, ini bisa menjadi gangguan visual bagi pengguna.

Jadi, bagaimana Anda akan menangani masalah ini?

Untuk menciptakan efek staggered, Anda dapat mengelompokkan kartu menggunakan komponen wrapper. Ini berarti `<SideNav>` statis akan ditampilkan terlebih dahulu, diikuti oleh kartu, dll.

Di file page.js Anda:

- Hapus komponen `<Card>` Anda.
- Hapus fungsi `fetchCardData()`.
- Impor komponen wrapper baru yang disebut `<CardWrapper />`.
- Impor komponen skeleton baru yang disebut `<CardsSkeleton />`.
- Bungkus `<CardWrapper />` dalam Suspense.

```javascript
// /app/dashboard/page.js
import CardWrapper from "@/app/ui/dashboard/cards";
// ...
import {
  RevenueChartSkeleton,
  LatestInvoicesSkeleton,
  CardsSkeleton,
} from "@/app/ui/skeletons";

export default async function Page() {
  return (
    <main>
      <h1 className={`${lusitana.className} mb-4 text-xl md:text-2xl`}>
        Dashboard
      </h1>
      <div className="grid gap-6 sm:grid-cols-2 lg:grid-cols-4">
        <Suspense fallback={<CardsSkeleton />}>
          <CardWrapper />
        </Suspense>
      </div>
      // ...
    </main>
  );
}
```

Kemudian, pindah ke file /app/ui/dashboard/cards.js, impor fungsi `fetchCardData()`, dan panggil di dalam komponen `<CardWrapper/>`. Pastikan untuk meng-uncomment kode yang diperlukan di komponen ini.

```javascript
// /app/ui/dashboard/cards.js
// ...
import { fetchCardData } from "@/app/lib/data";

// ...

export default async function CardWrapper() {
  const {
    numberOfInvoices,
    numberOfCustomers,
    totalPaidInvoices,
    totalPendingInvoices,
  } = await fetchCardData();

  return (
    <>
      <Card title="Collected" value={totalPaidInvoices} type="collected" />
      <Card title="Pending" value={totalPendingInvoices} type="pending" />
      <Card title="Total Invoices" value={numberOfInvoices} type="invoices" />
      <Card
        title="Total Customers"
        value={numberOfCustomers}
        type="customers"
      />
    </>
  );
}
```

Segarkan halaman, dan Anda seharusnya melihat semua kartu dimuat pada saat yang sama. Anda dapat menggunakan pola ini ketika Anda ingin beberapa komponen dimuat pada saat yang sama.

### Memutuskan Dimana Menempatkan Batas Suspense Anda

Dimana Anda menempatkan batas Suspense Anda akan bergantung pada beberapa hal:

- Bagaimana Anda ingin pengguna mengalami halaman saat dialirkan.
- Konten apa yang ingin Anda prioritaskan.
- Apakah komponen bergantung pada pengambilan data.

Lihatlah halaman dashboard Anda, adakah sesuatu yang akan Anda lakukan secara berbeda?

Jangan khawatir. Tidak ada jawaban yang benar.

- Anda dapat melakukan streaming seluruh halaman seperti yang kita lakukan dengan loading.js... tetapi itu dapat menyebabkan waktu muat lebih lama jika salah satu komponen memiliki pengambilan data yang lambat.
- Anda dapat melakukan streaming setiap komponen secara individu... tetapi itu dapat menyebabkan UI muncul di layar saat siap.
- Anda juga dapat menciptakan efek staggered dengan melakukan streaming bagian halaman. Tetapi Anda perlu membuat komponen wrapper.

Dimana Anda menempatkan batas suspense Anda akan bervariasi tergantung pada aplikasi Anda. Secara umum, adalah praktik yang baik untuk memindahkan pengambilan data Anda ke komponen yang membutuhkannya, dan kemudian membungkus komponen tersebut dalam Suspense. Tetapi tidak ada yang salah dengan melakukan streaming bagian atau seluruh halaman jika itu yang dibutuhkan oleh aplikasi Anda.

Jangan takut untuk bereksperimen dengan Suspense dan lihat apa yang terbaik, ini adalah API yang kuat yang dapat membantu Anda menciptakan pengalaman pengguna yang lebih menyenangkan.

### Melihat ke Depan

Streaming dan Komponen Server memberi kita cara baru untuk menangani pengambilan data dan status pemuatan, dengan tujuan akhir meningkatkan pengalaman pengguna.

Di bab berikutnya, Anda akan belajar tentang Partial Prerendering, model rendering Next.js baru yang dibangun dengan streaming dalam pikiran.
