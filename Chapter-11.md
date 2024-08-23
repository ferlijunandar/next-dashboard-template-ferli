# Bab 11

## Adding Search and Pagination

Pada bab sebelumnya, Anda meningkatkan performa pemuatan awal dashboard Anda dengan streaming. Sekarang mari kita lanjutkan ke halaman /invoices, dan pelajari cara menambahkan pencarian dan paginasi!

### Dalam bab ini...

Berikut adalah topik yang akan kita bahas:

- Pelajari cara menggunakan API Next.js: useSearchParams, usePathname, dan useRouter.
- Implementasi pencarian dan paginasi menggunakan parameter pencarian URL.

### Kode Awal

Di dalam file /dashboard/invoices/page.js Anda, tempelkan kode berikut:

```javascript
// /app/dashboard/invoices/page.js
import Pagination from "@/app/ui/invoices/pagination";
import Search from "@/app/ui/search";
import Table from "@/app/ui/invoices/table";
import { CreateInvoice } from "@/app/ui/invoices/buttons";
import { lusitana } from "@/app/ui/fonts";
import { InvoicesTableSkeleton } from "@/app/ui/skeletons";
import { Suspense } from "react";

export default async function Page() {
  return (
    <div className="w-full">
      <div className="flex w-full items-center justify-between">
        <h1 className={`${lusitana.className} text-2xl`}>Invoices</h1>
      </div>
      <div className="mt-4 flex items-center justify-between gap-2 md:mt-8">
        <Search placeholder="Search invoices..." />
        <CreateInvoice />
      </div>
      {/*  <Suspense key={query + currentPage} fallback={<InvoicesTableSkeleton />}>
        <Table query={query} currentPage={currentPage} />
      </Suspense> */}
      <div className="mt-5 flex w-full justify-center">
        {/* <Pagination totalPages={totalPages} /> */}
      </div>
    </div>
  );
}
```

Luangkan waktu untuk membiasakan diri dengan halaman dan komponen yang akan Anda kerjakan:

- `<Search/>` memungkinkan pengguna mencari faktur tertentu.
- `<Pagination/>` memungkinkan pengguna menavigasi antar halaman faktur.
- `<Table/>` menampilkan faktur.

Fungsi pencarian Anda akan mencakup klien dan server. Ketika pengguna mencari faktur di klien, parameter URL akan diperbarui, data akan diambil di server, dan tabel akan dirender ulang di server dengan data baru.

### Mengapa Menggunakan Parameter Pencarian URL?

Seperti disebutkan di atas, Anda akan menggunakan parameter pencarian URL untuk mengelola status pencarian. Pola ini mungkin baru jika Anda terbiasa melakukannya dengan status sisi klien.

Ada beberapa manfaat mengimplementasikan pencarian dengan parameter URL:

- **URL yang Dapat Ditandai dan Dibagikan**: Karena parameter pencarian ada di URL, pengguna dapat menandai keadaan saat ini dari aplikasi, termasuk kueri pencarian dan filter mereka, untuk referensi atau berbagi di masa mendatang.
- **Rendering Sisi Server dan Pemuatan Awal**: Parameter URL dapat langsung dikonsumsi di server untuk merender status awal, membuatnya lebih mudah untuk menangani rendering server.
- **Analitik dan Pelacakan**: Memiliki kueri pencarian dan filter langsung di URL memudahkan pelacakan perilaku pengguna tanpa memerlukan logika sisi klien tambahan.

### Menambahkan Fitur Pencarian

Ini adalah hook klien Next.js yang akan Anda gunakan untuk mengimplementasikan fungsi pencarian:

- `useSearchParams` - Memungkinkan Anda mengakses parameter dari URL saat ini. Misalnya, parameter pencarian untuk URL ini /dashboard/invoices?page=1&query=pending akan terlihat seperti ini: {page: '1', query: 'pending'}.
- `usePathname` - Memungkinkan Anda membaca pathname URL saat ini. Misalnya, untuk rute /dashboard/invoices, `usePathname` akan mengembalikan '/dashboard/invoices'.
- `useRouter` - Memungkinkan navigasi antara rute dalam komponen klien secara programatis. Ada beberapa metode yang bisa Anda gunakan.

Berikut adalah ikhtisar singkat dari langkah-langkah implementasi:

1. Menangkap input pengguna.
2. Memperbarui URL dengan parameter pencarian.
3. Menyinkronkan URL dengan bidang input.
4. Memperbarui tabel untuk mencerminkan kueri pencarian.

#### 1. Menangkap Input Pengguna

Buka Komponen `<Search>` (/app/ui/search.js), dan Anda akan melihat:

- `"use client"` - Ini adalah Komponen Klien, yang berarti Anda dapat menggunakan event listeners dan hooks.
- `<input>` - Ini adalah input pencarian.

Buat fungsi handleSearch baru, dan tambahkan listener onChange ke elemen `<input>`. `onChange` akan memanggil `handleSearch` setiap kali nilai input berubah.

```javascript
// /app/ui/search.js
"use client";

import { MagnifyingGlassIcon } from "@heroicons/react/24/outline";

export default function Search({ placeholder }: { placeholder: string }) {
  function handleSearch(term) {
    console.log(term);
  }

  return (
    <div className="relative flex flex-1 flex-shrink-0">
      <label htmlFor="search" className="sr-only">
        Search
      </label>
      <input
        className="peer block w-full rounded-md border border-gray-200 py-[9px] pl-10 text-sm outline-2 placeholder:text-gray-500"
        placeholder={placeholder}
        onChange={(e) => {
          handleSearch(e.target.value);
        }}
      />
      <MagnifyingGlassIcon className="absolute left-3 top-1/2 h-[18px] w-[18px] -translate-y-1/2 text-gray-500 peer-focus:text-gray-900" />
    </div>
  );
}
```

Uji apakah ini berfungsi dengan membuka konsol di Developer Tools, lalu ketikkan ke dalam field pencarian. Anda harus melihat istilah pencarian dicatat ke konsol.

Bagus! Anda menangkap input pencarian pengguna. Sekarang, Anda perlu memperbarui URL dengan istilah pencarian.

#### 2. Memperbarui URL dengan Parameter Pencarian

Impor hook `useSearchParams` dari 'next/navigation', dan tetapkan ke variabel:

```javascript
// /app/ui/search.js
"use client";

import { MagnifyingGlassIcon } from "@heroicons/react/24/outline";
import { useSearchParams } from "next/navigation";

export default function Search() {
  const searchParams = useSearchParams();

  function handleSearch(term: string) {
    console.log(term);
  }
  // ...
}
```

Di dalam `handleSearch`, buat instance `URLSearchParams` baru menggunakan variabel `searchParams` baru Anda.

```javascript
// /app/ui/search.js
"use client";

import { MagnifyingGlassIcon } from "@heroicons/react/24/outline";
import { useSearchParams } from "next/navigation";

export default function Search() {
  const searchParams = useSearchParams();

  function handleSearch(term: string) {
    const params = new URLSearchParams(searchParams);
  }
  // ...
}
```

`URLSearchParams` adalah API Web yang menyediakan metode utilitas untuk memanipulasi parameter kueri URL. Alih-alih membuat literal string yang rumit, Anda dapat menggunakannya untuk mendapatkan string params seperti ?page=1&query=a.

Selanjutnya, atur string params berdasarkan input pengguna. Jika input kosong, Anda ingin menghapusnya:

```javascript
// /app/ui/search.js
"use client";

import { MagnifyingGlassIcon } from "@heroicons/react/24/outline";
import { useSearchParams } from "next/navigation";

export default function Search() {
  const searchParams = useSearchParams();

  function handleSearch(term: string) {
    const params = new URLSearchParams(searchParams);
    if (term) {
      params.set("query", term);
    } else {
      params.delete("query");
    }
  }
  // ...
}
```

Sekarang Anda memiliki string kueri. Anda dapat menggunakan hook `useRouter` dan `usePathname` dari Next.js untuk memperbarui URL.

Impor `useRouter` dan `usePathname` dari 'next/navigation', dan gunakan metode `replace` dari `useRouter()` di dalam `handleSearch`:

```javascript
// /app/ui/search.js
"use client";

import { MagnifyingGlassIcon } from "@heroicons/react/24/outline";
import { useSearchParams, usePathname, useRouter } from "next/navigation";

export default function Search() {
  const searchParams = useSearchParams();
  const pathname = usePathname();
  const { replace } = useRouter();

  function handleSearch(term) {
    const params = new URLSearchParams(searchParams);
    if (term) {
      params.set("query", term);
    } else {
      params.delete("query");
    }
    replace(`${pathname}?${params.toString()}`);
  }
}
```

Berikut adalah penjelasan tentang apa yang terjadi:

- `${pathname}` adalah jalur saat ini, dalam kasus Anda, "/dashboard/invoices".
- Saat pengguna mengetik ke dalam bar pencarian, `params.toString()` menerjemahkan input ini ke dalam format yang ramah URL.
- `replace(${pathname}?${params.toString()})` memperbarui URL dengan data pencarian pengguna. Misalnya, /dashboard/invoices?

query=lee jika pengguna mencari "Lee".

- URL diperbarui tanpa memuat ulang halaman, berkat navigasi sisi klien Next.js (yang Anda pelajari di bab tentang navigasi antar halaman).

#### 3. Menyinkronkan URL dan Input

Untuk memastikan field input tetap sinkron dengan URL dan akan terisi saat dibagikan, Anda dapat meneruskan `defaultValue` ke input dengan membaca dari `searchParams`:

```javascript
// /app/ui/search.js
<input
  className="peer block w-full rounded-md border border-gray-200 py-[9px] pl-10 text-sm outline-2 placeholder:text-gray-500"
  placeholder={placeholder}
  onChange={(e) => {
    handleSearch(e.target.value);
  }}
  defaultValue={searchParams.get("query")?.toString()}
/>
```

`defaultValue` vs. `value` / Komponen Terkontrol vs. Tidak Terkontrol

Jika Anda menggunakan state untuk mengelola nilai input, Anda akan menggunakan atribut `value` untuk menjadikannya komponen terkontrol. Ini berarti React akan mengelola state input.

Namun, karena Anda tidak menggunakan state, Anda dapat menggunakan `defaultValue`. Ini berarti input asli akan mengelola state-nya sendiri. Ini baik-baik saja karena Anda menyimpan kueri pencarian ke URL alih-alih state.

#### 4. Memperbarui Tabel

Terakhir, Anda perlu memperbarui komponen tabel untuk mencerminkan kueri pencarian.

Navigasikan kembali ke halaman faktur.

Komponen halaman menerima prop bernama `searchParams`, jadi Anda dapat meneruskan parameter URL saat ini ke komponen `<Table>`.

```javascript
// /app/dashboard/invoices/page.js
import Pagination from "@/app/ui/invoices/pagination";
import Search from "@/app/ui/search";
import Table from "@/app/ui/invoices/table";
import { CreateInvoice } from "@/app/ui/invoices/buttons";
import { lusitana } from "@/app/ui/fonts";
import { Suspense } from "react";
import { InvoicesTableSkeleton } from "@/app/ui/skeletons";

export default async function Page({ searchParams }) {
  const query = searchParams?.query || "";
  const currentPage = Number(searchParams?.page) || 1;

  return (
    <div className="w-full">
      <div className="flex w-full items-center justify-between">
        <h1 className={`${lusitana.className} text-2xl`}>Invoices</h1>
      </div>
      <div className="mt-4 flex items-center justify-between gap-2 md:mt-8">
        <Search placeholder="Search invoices..." />
        <CreateInvoice />
      </div>
      <Suspense key={query + currentPage} fallback={<InvoicesTableSkeleton />}>
        <Table query={query} currentPage={currentPage} />
      </Suspense>
      <div className="mt-5 flex w-full justify-center">
        {/* <Pagination totalPages={totalPages} /> */}
      </div>
    </div>
  );
}
```

Jika Anda menavigasi ke Komponen `<Table>`, Anda akan melihat bahwa dua prop, `query` dan `currentPage`, diteruskan ke fungsi `fetchFilteredInvoices()` yang mengembalikan faktur yang cocok dengan kueri.

```javascript
// /app/ui/invoices/table.js
// ...
export default async function InvoicesTable({ query, currentPage }) {
  const invoices = await fetchFilteredInvoices(query, currentPage);
  // ...
}
```

Dengan perubahan ini, silakan uji coba. Jika Anda mencari istilah, URL akan diperbarui, yang akan mengirim permintaan baru ke server, data akan diambil di server, dan hanya faktur yang sesuai dengan kueri Anda yang akan dikembalikan.

### Kapan Menggunakan Hook `useSearchParams()` vs. Prop `searchParams`?

Anda mungkin telah memperhatikan bahwa Anda menggunakan dua cara berbeda untuk mengekstrak parameter pencarian. Apakah Anda menggunakan yang satu atau yang lain tergantung pada apakah Anda bekerja di klien atau server.

- `<Search>` adalah Komponen Klien, jadi Anda menggunakan hook `useSearchParams()` untuk mengakses parameter dari klien.
- `<Table>` adalah Komponen Server yang mengambil datanya sendiri, jadi Anda dapat meneruskan prop `searchParams` dari halaman ke komponen.

Sebagai aturan umum, jika Anda ingin membaca parameter dari klien, gunakan hook `useSearchParams()` karena ini menghindari harus kembali ke server.

### Praktik Terbaik: Debouncing

Selamat! Anda telah mengimplementasikan pencarian dengan Next.js! Tetapi ada sesuatu yang bisa Anda lakukan untuk mengoptimalkannya.

Di dalam fungsi `handleSearch` Anda, tambahkan `console.log` berikut:

```javascript
// /app/ui/search.js
function handleSearch(term) {
  console.log(`Searching... ${term}`);

  const params = new URLSearchParams(searchParams);
  if (term) {
    params.set("query", term);
  } else {
    params.delete("query");
  }
  replace(`${pathname}?${params.toString()}`);
}
```

Kemudian ketik "Emil" ke dalam bilah pencarian Anda dan periksa konsol di alat pengembang. Apa yang terjadi?

```
Dev Tools Console

Searching... E
Searching... Em
Searching... Emi
Searching... Emil
```

Anda memperbarui URL pada setiap penekanan tombol, dan karenanya mengkueri database Anda pada setiap penekanan tombol! Ini bukan masalah karena aplikasi kita kecil, tetapi bayangkan jika aplikasi Anda memiliki ribuan pengguna, masing-masing mengirimkan permintaan baru ke database Anda pada setiap penekanan tombol.

Debouncing adalah praktik pemrograman yang membatasi laju di mana suatu fungsi dapat dipicu. Dalam kasus kita, Anda hanya ingin mengkueri database ketika pengguna telah berhenti mengetik.

### Cara Kerja Debouncing:

- **Trigger Event**: Ketika sebuah acara yang harus dibatasi (seperti penekanan tombol di kotak pencarian) terjadi, sebuah timer dimulai.
- **Wait**: Jika sebuah acara baru terjadi sebelum timer berakhir, timer diatur ulang.
- **Execution**: Jika timer mencapai akhir hitung mundurnya, fungsi yang dibatasi dijalankan.

Anda dapat mengimplementasikan debouncing dengan beberapa cara, termasuk secara manual membuat fungsi debounce Anda sendiri. Untuk menjaga kesederhanaan, kita akan menggunakan perpustakaan yang disebut `use-debounce`.

Instal `use-debounce`:

```shell
pnpm i use-debounce
```

Di dalam Komponen `<Search>`, impor fungsi yang disebut `useDebouncedCallback`:

```javascript
// /app/ui/search.js
// ...
import { useDebouncedCallback } from "use-debounce";

// Di dalam Komponen Search...
const handleSearch = useDebouncedCallback((term) => {
  console.log(`Searching... ${term}`);

  const params = new URLSearchParams(searchParams);
  if (term) {
    params.set("query", term);
  } else {
    params.delete("query");
  }
  replace(`${pathname}?${params.toString()}`);
}, 300);
```

Fungsi ini akan membungkus konten `handleSearch`, dan hanya menjalankan kode setelah waktu tertentu begitu pengguna berhenti mengetik (300ms).

Sekarang ketikkan di bilah pencarian Anda lagi, dan buka konsol di alat pengembang. Anda harus melihat yang berikut:

```
Dev Tools Console

Searching... Emil
```

Dengan debouncing, Anda dapat mengurangi jumlah permintaan yang dikirim ke database Anda, sehingga menghemat sumber daya.

### Menambahkan Paginasi

Setelah memperkenalkan fitur pencarian, Anda akan melihat tabel hanya menampilkan 6 faktur sekaligus. Ini karena fungsi `fetchFilteredInvoices()` di data.ts mengembalikan maksimal 6 faktur per halaman.

Menambahkan paginasi memungkinkan pengguna untuk menavigasi melalui halaman yang berbeda untuk melihat semua faktur. Mari kita lihat bagaimana Anda dapat mengimplementasikan paginasi menggunakan parameter URL, seperti yang Anda lakukan dengan pencarian.

Navigasikan ke komponen `<Pagination>` dan Anda akan melihat bahwa ini adalah Komponen Klien. Anda tidak ingin mengambil data di klien karena ini akan mengekspos rahasia database Anda (ingat, Anda tidak menggunakan lapisan API). Sebaliknya, Anda dapat mengambil data di server, dan meneruskannya ke komponen sebagai prop.

Di /dashboard/invoices/page.js, impor fungsi baru yang disebut `fetchInvoicesPages` dan teruskan kueri dari `searchParams` sebagai argumen:

```javascript
// /app/dashboard/invoices/page.js
// ...
import { fetchInvoicesPages } from '@/app/lib/data';

export default async function Page(searchParams) {
  const query = searchParams?.query || '';
  const currentPage = Number(searchParams?.page) || 1;

  const totalPages = await fetchInvoicesPages(query);

  return (
    // ...
  );
}
```

`fetchInvoicesPages` mengembalikan jumlah total halaman berdasarkan kueri pencarian. Misalnya, jika ada 12 faktur yang cocok dengan kueri pencarian, dan setiap halaman menampilkan 6 faktur, maka jumlah total halaman adalah 2.

Selanjutnya, teruskan prop `totalPages` ke komponen `<Pagination>`:

```javascript
// /app/dashboard/invoices/page.js
// ...

export default async function Page(searchParams) {
  const query = searchParams?.query || "";
  const currentPage = Number(searchParams?.page) || 1;

  const totalPages = await fetchInvoicesPages(query);

  return (
    <div className="w-full">
      <div className="flex w-full items-center justify-between">
        <h1 className={`${lusitana.className} text-2xl`}>Invoices</h1>
      </div>
      <div className="mt-4 flex items-center justify-between gap-2 md:mt-8">
        <Search placeholder="Search invoices..." />
        <CreateInvoice />
      </div>
      <Suspense key={query + currentPage} fallback={<InvoicesTableSkeleton />}>
        <Table query={query} currentPage={currentPage} />
      </Suspense>
      <div className="mt-5 flex w-full justify-center">
        <Pagination totalPages={totalPages} />
      </div>
    </div>
  );
}
```

Navigasikan ke komponen `<Pagination>` dan impor hook `usePathname` dan `useSearchParams`. Kami akan menggunakan ini untuk mendapatkan halaman saat ini dan mengatur halaman baru. Pastikan juga untuk meng-uncomment kode di komponen ini. Aplikasi Anda akan rusak sementara karena Anda belum mengimplementasikan logika `<Pagination>`. Mari kita lakukan sekarang!

```javascript
// /app/ui/invoices/pagination.js
"use client";

import { ArrowLeftIcon, ArrowRightIcon } from "@heroicons/react/24/outline";
import clsx from "clsx";
import Link from "next/link";
import { generatePagination } from "@/app/lib/utils";
import { usePathname, useSearchParams } from "next/navigation";

export default function Pagination(totalPages) {
  const pathname = usePathname();
  const searchParams = useSearchParams();
  const currentPage = Number(searchParams.get("page")) || 1;

  // ...
}
```

Selanjutnya, buat fungsi baru di dalam Komponen `<Pagination>` yang disebut `createPageURL`. Sama seperti pencarian, Anda akan menggunakan `URLSearchParams` untuk mengatur nomor halaman baru, dan `pathName` untuk membuat string URL.

```javascript
// /app/ui/invoices/pagination.js
"use client";

import { ArrowLeftIcon, ArrowRightIcon } from "@heroicons/react/24/outline";
import clsx from "clsx";
import Link from "next/link";
import { generatePagination } from "@/app/lib/utils";
import { usePathname, useSearchParams } from "next/navigation";

export default function Pagination(totalPages) {
  const pathname = usePathname();
  const searchParams = useSearchParams();
  const currentPage = Number(searchParams.get("page")) || 1;

  const createPageURL = (pageNumber) => {
    const params = new URLSearchParams(searchParams);
    params.set("page", pageNumber.toString());
    return `${pathname}?${params.toString()}`;
  };

  // ...
}
```

Berikut adalah penjelasan tentang apa yang terjadi:

- `createPageURL` membuat instance dari parameter pencarian saat ini.
- Kemudian, memperbarui parameter "page" ke nomor halaman yang diberikan.
- Terakhir, menyusun URL lengkap menggunakan `pathname` dan parameter pencarian yang diperbarui.

Sisa dari komponen `<Pagination>` berurusan dengan styling dan status yang berbeda (pertama, terakhir, aktif, dinonaktifkan, dll). Kami tidak akan membahas detailnya dalam kursus ini, tetapi jangan ragu untuk melihat kode untuk melihat di mana `createPageURL` dipanggil.

Terakhir, ketika pengguna mengetikkan kueri pencarian baru, Anda ingin mengatur ulang nomor halaman ke 1. Anda dapat melakukannya dengan memperbarui fungsi `handleSearch` di komponen `<Search>` Anda:

```javascript
// /app/ui/search.js
"use client";

import { MagnifyingGlassIcon } from "@heroicons/react/24/outline";
import { usePathname, useRouter, useSearchParams } from "next/navigation";
import { useDebouncedCallback } from "use-debounce";

export default function Search(placeholder) {
  const searchParams = useSearchParams();
  const { replace } = useRouter();
  const pathname = usePathname();

  const handleSearch = useDebouncedCallback((term) => {
    const params = new URLSearchParams(searchParams);
    params.set("page", "1");
    if (term) {
      params.set("query", term);
    } else {
      params.delete("query");
    }
    replace(`${pathname}?${params.toString()}`);
  }, 300);
}
```

### Ringkasan

Selamat! Anda baru saja mengimplementasikan pencarian dan paginasi menggunakan URL Params dan API Next.js.

Untuk merangkum, dalam bab ini:

- Anda menangani pencarian dan paginasi dengan parameter pencarian URL alih-alih status klien.
- Anda mengambil data di server.
- Anda menggunakan hook router `useRouter` untuk transisi sisi klien yang lebih halus.

Pola-pola ini berbeda dari apa yang mungkin Anda biasa lakukan saat bekerja dengan React sisi klien, tetapi semoga sekarang Anda lebih memahami manfaat menggunakan parameter pencarian URL dan mengangkat status ini ke server.
