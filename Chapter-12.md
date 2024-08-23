# Bab 12

## Mutating Data

Pada bab sebelumnya, Anda mengimplementasikan pencarian dan paginasi menggunakan URL Search Params dan API Next.js. Mari kita lanjutkan bekerja pada halaman Invoices dengan menambahkan kemampuan untuk membuat, memperbarui, dan menghapus faktur!

### Dalam bab ini...

Berikut adalah topik yang akan kita bahas:

- Apa itu React Server Actions dan bagaimana menggunakannya untuk memutasi data.
- Cara bekerja dengan formulir dan Komponen Server.
- Praktik terbaik untuk bekerja dengan objek formData asli, termasuk validasi tipe.
- Cara merevalidasi cache klien menggunakan API revalidatePath.
- Cara membuat segmen rute dinamis dengan ID spesifik.

### Apa itu Server Actions?

React Server Actions memungkinkan Anda menjalankan kode asinkron langsung di server. Mereka menghilangkan kebutuhan untuk membuat endpoint API untuk memutasi data Anda. Sebaliknya, Anda menulis fungsi asinkron yang dieksekusi di server dan dapat dipanggil dari Komponen Klien atau Server.

Keamanan adalah prioritas utama untuk aplikasi web, karena mereka rentan terhadap berbagai ancaman. Di sinilah Server Actions berperan. Mereka menawarkan solusi keamanan yang efektif, melindungi dari berbagai jenis serangan, mengamankan data Anda, dan memastikan akses yang sah. Server Actions mencapai ini melalui teknik seperti permintaan POST, penutupan terenkripsi, pemeriksaan input ketat, hashing pesan kesalahan, dan pembatasan host, yang semuanya bekerja bersama untuk secara signifikan meningkatkan keamanan aplikasi Anda.

### Menggunakan formulir dengan Server Actions

Dalam React, Anda dapat menggunakan atribut action dalam elemen `<form>` untuk memanggil actions. Action akan secara otomatis menerima objek FormData asli, yang berisi data yang ditangkap.

Contohnya:

```javascript
// Komponen Server
export default function Page() {
  // Action
  async function create(formData: FormData) {
    "use server";

    // Logika untuk memutasi data...
  }

  // Panggil action menggunakan atribut "action"
  return <form action={create}>...</form>;
}
```

Keuntungan memanggil Server Action dalam Komponen Server adalah peningkatan progresif - formulir bekerja bahkan jika JavaScript dinonaktifkan di klien.

### Next.js dengan Server Actions

Server Actions juga terintegrasi erat dengan caching Next.js. Ketika sebuah formulir dikirim melalui Server Action, Anda tidak hanya dapat menggunakan action untuk memutasi data, tetapi juga dapat merevalidasi cache terkait menggunakan API seperti revalidatePath dan revalidateTag.

Mari kita lihat bagaimana semuanya bekerja bersama!

### Membuat faktur

Berikut adalah langkah-langkah yang akan Anda ambil untuk membuat faktur baru:

1. Membuat formulir untuk menangkap input pengguna.
2. Membuat Server Action dan memanggilnya dari formulir.
3. Di dalam Server Action Anda, ekstrak data dari objek formData.
4. Validasi dan persiapkan data untuk dimasukkan ke dalam database Anda.
5. Masukkan data dan tangani kesalahan apa pun.
6. Revalidasi cache dan arahkan kembali pengguna ke halaman faktur.

#### 1. Membuat rute baru dan formulir

Untuk memulai, di dalam folder /invoices, tambahkan segmen rute baru yang disebut /create dengan file page.js:
![alt text](image-28.png)
Folder Invoices dengan folder create tersemat, dan file page.js di dalamnya
Anda akan menggunakan rute ini untuk membuat faktur baru. Di dalam file page.js Anda, tempelkan kode berikut, lalu luangkan waktu untuk mempelajarinya:

```javascript
// /dashboard/invoices/create/page.js
import Form from "@/app/ui/invoices/create-form";
import Breadcrumbs from "@/app/ui/invoices/breadcrumbs";
import { fetchCustomers } from "@/app/lib/data";

export default async function Page() {
  const customers = await fetchCustomers();

  return (
    <main>
      <Breadcrumbs
        breadcrumbs={[
          { label: "Invoices", href: "/dashboard/invoices" },
          {
            label: "Create Invoice",
            href: "/dashboard/invoices/create",
            active: true,
          },
        ]}
      />
      <Form customers={customers} />
    </main>
  );
}
```

Halaman Anda adalah Komponen Server yang mengambil pelanggan dan meneruskannya ke komponen `<Form>`. Untuk menghemat waktu, kami telah membuat komponen `<Form>` untuk Anda.

Navigasikan ke komponen `<Form>`, dan Anda akan melihat bahwa formulir:

- Memiliki satu elemen `<select>` (dropdown) dengan daftar pelanggan.
- Memiliki satu elemen `<input>` untuk jumlah dengan type="number".
- Memiliki dua elemen `<input>` untuk status dengan type="radio".
- Memiliki satu tombol dengan type="submit".

Di http://localhost:3000/dashboard/invoices/create, Anda harus melihat UI berikut:
![alt text](image-29.png)
Halaman membuat faktur dengan breadcrumbs dan formulir

#### 2. Membuat Server Action

Bagus, sekarang mari kita buat Server Action yang akan dipanggil ketika formulir dikirimkan.

Navigasikan ke direktori lib Anda dan buat file baru bernama actions.js. Di bagian atas file ini, tambahkan direktif use server:

```javascript
// /app/lib/actions.js
"use server";
```

Dengan menambahkan 'use server', Anda menandai semua fungsi yang diekspor dalam file sebagai Server Actions. Fungsi-fungsi server ini kemudian dapat diimpor dan digunakan dalam komponen Klien dan Server.

Anda juga dapat menulis Server Actions langsung di dalam Komponen Server dengan menambahkan "use server" di dalam action. Tetapi untuk kursus ini, kita akan menyimpan semuanya terorganisir dalam file terpisah.

Di file actions.js Anda, buat fungsi asinkron baru yang menerima formData:

```javascript
// /app/lib/actions.js
"use server";

export async function createInvoice(formData) {}
```

Kemudian, di komponen `<Form>` Anda, impor `createInvoice` dari file actions.js Anda. Tambahkan atribut action ke elemen `<form>`, dan panggil action `createInvoice`.

```javascript
// /app/ui/invoices/create-form.js
import { customerField } from '@/app/lib/definitions';
import Link from 'next/link';
import {
  CheckIcon,
  ClockIcon,
  CurrencyDollarIcon,
  UserCircleIcon,
} from '@heroicons/react/24/outline';
import { Button } from '@/app/ui/button';
import { createInvoice } from '@/app/lib/actions';

export default function Form({customers}) {
  return (
    <form action={createInvoice}>
      // ...
  )
}
```

Hal yang perlu diketahui: Dalam HTML, Anda akan meneruskan URL ke atribut action. URL ini adalah tujuan di mana data formulir Anda harus dikirimkan (biasanya endpoint API).

Namun, dalam React, atribut action dianggap sebagai prop khusus - artinya React membangun di atasnya untuk memungkinkan action dipanggil.

Di belakang layar, Server Actions membuat endpoint API POST. Inilah sebabnya Anda tidak perlu membuat endpoint API secara manual saat menggunakan Server Actions.

#### 3. Ekstrak data dari formData

Kembali di file actions.js Anda, Anda perlu mengekstrak nilai formData, ada beberapa metode yang dapat Anda gunakan. Untuk contoh ini, mari kita gunakan metode `.get(name)`.

```javascript
// /app/lib/actions.js
"use server";

export async function createInvoice(formData) {
  const rawFormData = {
    customerId: formData.get("customerId"),
    amount: formData.get("amount"),
    status: formData.get("status"),
  };
  // Coba:
  console.log(rawFormData);
}
```

Tip: Jika Anda bekerja dengan formulir yang memiliki banyak field, Anda mungkin ingin mempertimbangkan untuk menggunakan metode entries() dengan Object.fromEntries() dari JavaScript. Contohnya:

```javascript
const rawFormData = Object.fromEntries(formData.entries());
```

Untuk memeriksa apakah semuanya terhubung dengan benar, coba kirimkan formulir. Setelah dikirimkan, Anda harus melihat data yang baru saja Anda masukkan ke dalam formulir tercatat di terminal Anda.

Sekarang data Anda dalam bentuk objek, akan jauh lebih mudah untuk dikerjakan.

#### 4. Validasi dan persiapkan data

Sebelum mengirimkan data formulir ke database Anda, Anda ingin memastikan bahwa data tersebut dalam format yang benar dan dengan tipe yang benar. Jika Anda ingat dari sebelumnya dalam kursus, tabel faktur Anda mengharapkan data dalam format berikut:

```javascript
// /app/lib/definitions.js
export type Invoice = {
  id: string, // Akan dibuat di database
  customer_id: string,
  amount: number, // Disimpan dalam sen
  status: "pending" | "paid",
  date: string,
};
```

Sejauh ini, Anda hanya memiliki customer_id, amount, dan status dari formulir.

##### Validasi tipe dan koersi

Penting untuk memvalidasi bahwa data dari formulir Anda sesuai dengan tipe yang diharapkan di database Anda. Misalnya, jika Anda menambahkan `console.log` di dalam action Anda:

```javascript
console.log(typeof rawFormData.amount);
```

Anda akan melihat bahwa `amount` adalah tipe string, bukan number. Ini karena elemen input dengan type="number" sebenarnya mengembalikan string, bukan number!

Untuk menangani validasi tipe, Anda memiliki beberapa opsi. Meskipun Anda dapat memvalidasi tipe secara manual, menggunakan pustaka validasi tipe dapat menghemat waktu dan usaha Anda. Untuk contoh Anda, kita akan menggunakan Zod, pustaka validasi TypeScript-first yang dapat

menyederhanakan tugas ini untuk Anda.

Di file actions.js Anda, impor Zod dan definisikan skema yang sesuai dengan bentuk objek formulir Anda. Skema ini akan memvalidasi formData sebelum menyimpannya ke database.

```javascript
// /app/lib/actions.js
"use server";

import { z } from "zod";

const FormSchema = z.object({
  id: z.string(),
  customerId: z.string(),
  amount: z.coerce.number(),
  status: z.enum(["pending", "paid"]),
  date: z.string(),
});

const CreateInvoice = FormSchema.omit({ id: true, date: true });

export async function createInvoice(formData) {
  // ...
}
```

Field `amount` secara khusus diatur untuk mengubah (mengubah) dari string ke number sambil juga memvalidasi tipe-nya.

Anda kemudian dapat meneruskan `rawFormData` Anda ke `CreateInvoice` untuk memvalidasi tipe:

```javascript
// /app/lib/actions.js
// ...
export async function createInvoice(formData) {
  const { customerId, amount, status } = CreateInvoice.parse({
    customerId: formData.get("customerId"),
    amount: formData.get("amount"),
    status: formData.get("status"),
  });
}
```

##### Menyimpan nilai dalam sen

Biasanya praktik yang baik untuk menyimpan nilai moneter dalam sen di database Anda untuk menghilangkan kesalahan floating-point JavaScript dan memastikan akurasi yang lebih besar.

Mari kita konversi `amount` menjadi sen:

```javascript
// /app/lib/actions.js
// ...
export async function createInvoice(formData) {
  const { customerId, amount, status } = CreateInvoice.parse({
    customerId: formData.get("customerId"),
    amount: formData.get("amount"),
    status: formData.get("status"),
  });
  const amountInCents = amount * 100;
}
```

##### Membuat tanggal baru

Terakhir, mari kita buat tanggal baru dengan format "YYYY-MM-DD" untuk tanggal pembuatan faktur:

```javascript
// /app/lib/actions.js
// ...
export async function createInvoice(formData) {
  const { customerId, amount, status } = CreateInvoice.parse({
    customerId: formData.get("customerId"),
    amount: formData.get("amount"),
    status: formData.get("status"),
  });
  const amountInCents = amount * 100;
  const date = new Date().toISOString().split("T")[0];
}
```

#### 5. Memasukkan data ke database Anda

Sekarang setelah Anda memiliki semua nilai yang Anda butuhkan untuk database Anda, Anda dapat membuat kueri SQL untuk memasukkan faktur baru ke database Anda dan memasukkan variabelnya:

```javascript
// /app/lib/actions.js
import { z } from "zod";
import { sql } from "@vercel/postgres";

// ...

export async function createInvoice(formData) {
  const { customerId, amount, status } = CreateInvoice.parse({
    customerId: formData.get("customerId"),
    amount: formData.get("amount"),
    status: formData.get("status"),
  });
  const amountInCents = amount * 100;
  const date = new Date().toISOString().split("T")[0];

  await sql`
    INSERT INTO invoices (customer_id, amount, status, date)
    VALUES (${customerId}, ${amountInCents}, ${status}, ${date})
  `;
}
```

Saat ini, kita tidak menangani kesalahan apa pun. Kita akan melakukannya di bab berikutnya. Untuk sekarang, mari kita lanjutkan ke langkah berikutnya.

#### 6. Merevalidasi dan mengarahkan ulang

Next.js memiliki Client-side Router Cache yang menyimpan segmen rute di browser pengguna untuk waktu tertentu. Bersama dengan prefetching, cache ini memastikan bahwa pengguna dapat dengan cepat menavigasi antara rute sambil mengurangi jumlah permintaan yang dibuat ke server.

Karena Anda memperbarui data yang ditampilkan di rute faktur, Anda ingin menghapus cache ini dan memicu permintaan baru ke server. Anda dapat melakukannya dengan fungsi `revalidatePath` dari Next.js:

```javascript
// /app/lib/actions.js
"use server";

import { z } from "zod";
import { sql } from "@vercel/postgres";
import { revalidatePath } from "next/cache";

// ...

export async function createInvoice(formData) {
  const { customerId, amount, status } = CreateInvoice.parse({
    customerId: formData.get("customerId"),
    amount: formData.get("amount"),
    status: formData.get("status"),
  });
  const amountInCents = amount * 100;
  const date = new Date().toISOString().split("T")[0];

  await sql`
    INSERT INTO invoices (customer_id, amount, status, date)
    VALUES (${customerId}, ${amountInCents}, ${status}, ${date})
  `;

  revalidatePath("/dashboard/invoices");
}
```

Setelah database diperbarui, jalur `/dashboard/invoices` akan divalidasi ulang, dan data baru akan diambil dari server.

Pada titik ini, Anda juga ingin mengarahkan pengguna kembali ke halaman `/dashboard/invoices`. Anda dapat melakukannya dengan fungsi `redirect` dari Next.js:

```javascript
// /app/lib/actions.js
"use server";

import { z } from "zod";
import { sql } from "@vercel/postgres";
import { revalidatePath } from "next/cache";
import { redirect } from "next/navigation";

// ...

export async function createInvoice(formData) {
  // ...

  revalidatePath("/dashboard/invoices");
  redirect("/dashboard/invoices");
}
```

Selamat! Anda baru saja mengimplementasikan Server Action pertama Anda. Uji coba dengan menambahkan faktur baru, jika semuanya bekerja dengan benar:

- Anda harus diarahkan ke rute `/dashboard/invoices` saat pengiriman.
- Anda harus melihat faktur baru di bagian atas tabel.

### Memperbarui faktur

Formulir memperbarui faktur mirip dengan formulir membuat faktur, kecuali Anda perlu meneruskan ID faktur untuk memperbarui catatan di database Anda. Mari kita lihat bagaimana Anda bisa mendapatkan dan meneruskan ID faktur.

Berikut adalah langkah-langkah yang akan Anda ambil untuk memperbarui faktur:

1. Membuat segmen rute dinamis dengan ID faktur.
2. Membaca ID faktur dari parameter halaman.
3. Mengambil faktur spesifik dari database Anda.
4. Memperbarui formulir dengan data faktur.
5. Memperbarui data faktur di database Anda.

#### 1. Membuat Segmen Rute Dinamis dengan ID faktur

Next.js memungkinkan Anda membuat Segmen Rute Dinamis ketika Anda tidak tahu nama segmen yang tepat dan ingin membuat rute berdasarkan data. Ini bisa berupa judul posting blog, halaman produk, dll. Anda dapat membuat segmen rute dinamis dengan membungkus nama folder dalam tanda kurung siku. Misalnya, [id], [post], atau [slug].

Di folder /invoices Anda, buat rute dinamis baru yang disebut [id], lalu rute baru yang disebut edit dengan file page.js. Struktur file Anda harus terlihat seperti ini:
![alt text](image-30.png)
Folder Invoices dengan folder [id] yang disematkan, dan folder edit di dalamnya
Di komponen `<Table>` Anda, perhatikan bahwa ada tombol `<UpdateInvoice />` yang menerima ID faktur dari catatan tabel.

```javascript
// /app/ui/invoices/table.js
export default async function InvoicesTable({ query, currentPage }) {
  return (
    // ...
    <td className="flex justify-end gap-2 whitespace-nowrap px-6 py-4 text-sm">
      <UpdateInvoice id={invoice.id} />
      <DeleteInvoice id={invoice.id} />
    </td>
    // ...
  );
}
```

Navigasikan ke komponen `<UpdateInvoice />` Anda, dan perbarui href dari Link untuk menerima prop id. Anda dapat menggunakan template literals untuk menautkan ke segmen rute dinamis:

```javascript
// /app/ui/invoices/buttons.js
import { PencilIcon, PlusIcon, TrashIcon } from "@heroicons/react/24/outline";
import Link from "next/link";

// ...

export function UpdateInvoice({ id }) {
  return (
    <Link
      href={`/dashboard/invoices/${id}/edit`}
      className="rounded-md border p-2 hover:bg-gray-100"
    >
      <PencilIcon className="w-5" />
    </Link>
  );
}
```

#### 2. Membaca ID faktur dari parameter halaman

Kembali ke komponen `<Page>` Anda, tempelkan kode berikut:

```javascript
// /app/dashboard/invoices/[id]/edit/page.js
import Form from "@/app/ui/invoices/edit-form";
import Breadcrumbs from "@/app/ui/invoices/breadcrumbs";
import { fetchCustomers } from "@/app/lib/data";

export default async function Page() {
  return (
    <main>
      <Breadcrumbs
        breadcrumbs={[
          { label: "Invoices", href: "/dashboard/invoices" },
          {
            label: "Edit Invoice",
            href: `/dashboard/invoices/${id}/edit`,
            active: true,
          },
        ]}
      />

      <Form invoice={invoice} customers={customers} />
    </main>
  );
}
```

Perhatikan bahwa ini mirip dengan halaman /create invoice Anda, kecuali itu mengimpor formulir yang berbeda (dari file edit-form.js). Formulir ini harus diisi sebelumnya dengan `defaultValue` untuk nama pelanggan, jumlah faktur, dan status. Untuk mengisi field formulir, Anda perlu mengambil faktur spesifik menggunakan id.

Selain searchParams, komponen halaman juga menerima prop yang disebut params yang dapat Anda gunakan untuk mengakses id. Perbarui komponen `<Page>` Anda untuk menerima prop:

```javascript
// /app/dashboard/invoices/[id]/edit/page.js
import Form from "@/app/ui/invoices/edit-form";
import Breadcrumbs from "@/app/ui/invoices/breadcrumbs";
import { fetchCustomers } from "@/app/lib/data";

export default async function Page({ params }) {
  const id = params.id;
  // ...
}
```

#### 3. Mengambil faktur spesifik

Kemudian:

- Impor fungsi baru yang disebut `fetchInvoiceById` dan pass id sebagai argumen.
- Impor `fetchCustomers` untuk mengambil nama pelanggan untuk dropdown.
- Anda dapat menggunakan `Promise.all` untuk mengambil faktur dan pelanggan secara paralel:

```javascript
// /dashboard/invoices/[id]/edit/page.js
import Form from "@/app/ui/invoices/edit-form";
import Breadcrumbs from "@/app/ui/invoices/breadcrumbs";
import { fetchInvoiceById, fetchCustomers } from "@/app/lib/data";

export default async function Page({ params }) {
  const id = params.id;
  const [invoice, customers] = await Promise.all([
    fetchInvoiceById(id),
    fetchCustomers(),
  ]);
  // ...
}
```

Anda akan melihat kesalahan TS sementara untuk prop invoice di terminal Anda karena invoice berpotensi undefined. Jangan khawatir tentang itu untuk sekarang, Anda akan menyelesaikannya di bab berikutnya saat Anda menambahkan penanganan kesalahan.

Bagus! Sekarang, uji bahwa semuanya terhubung dengan benar. Kunjungi http://localhost:3000/dashboard/invoices dan klik ikon Pensil untuk mengedit faktur. Setelah navigasi, Anda harus melihat formulir yang diisi sebelumnya dengan detail faktur:
![alt text](image-31.png)
Halaman mengedit faktur dengan breadcrumbs dan formulir
URL juga harus diperbarui dengan ID sebagai berikut: http://localhost:3000/dashboard/invoice/uuid/edit

UUID vs. Kunci Auto-incrementing

Kami menggunakan UUID alih-alih kunci incrementing (misalnya, 1, 2, 3, dll.). Ini membuat URL lebih panjang; namun, UUID menghilangkan risiko tabrakan ID, unik secara global, dan mengurangi risiko serangan enumerasi - menjadikannya ideal untuk database besar.

Namun, jika Anda lebih suka URL yang lebih bersih, Anda mungkin lebih suka menggunakan kunci auto-incrementing.

#### 4. Pass ID ke Server Action

Terakhir, Anda ingin pass ID ke Server Action sehingga Anda dapat memperbarui catatan yang benar di database Anda. Anda tidak dapat meneruskan ID sebagai argumen seperti ini:

```javascript
// /app/ui/invoices/edit-form.js
// Passing an id as argument won't work
<form action={updateInvoice(id)}>
```

Sebagai gantinya, Anda dapat meneruskan ID ke Server Action menggunakan JS bind. Ini akan memastikan bahwa nilai apa pun yang diteruskan ke Server Action dienkripsi.

```javascript
// /app/ui/invoices/edit-form.js
// ...
import { updateInvoice } from "@/app/lib/actions";

export default function EditInvoiceForm({ invoice, customers }) {
  const updateInvoiceWithId = updateInvoice.bind(null, invoice.id);

  return (
    <form action={updateInvoiceWithId}>
      <input type="hidden" name="id" value={invoice.id} />
    </form>
  );
}
```

Catatan: Menggunakan field input tersembunyi dalam formulir Anda juga berfungsi (misalnya, `<input type="hidden" name="id" value={invoice.id} />`). Namun, nilai-nilai akan muncul sebagai teks lengkap dalam sumber HTML, yang tidak ideal untuk data sensitif seperti ID.

Kemudian, di file actions.js Anda, buat action baru, `updateInvoice`:

```javascript
// /app/lib/actions.js
// Gunakan Zod untuk memperbarui tipe yang diharapkan
const UpdateInvoice = FormSchema.omit({ id: true, date: true });

// ...

export async function updateInvoice(id, formData) {
  const { customerId, amount, status } = UpdateInvoice.parse({
    customerId: formData.get("customerId"),
    amount: formData.get("amount"),
    status: formData.get("status"),
  });

  const amountInCents = amount * 100;

  await sql`
    UPDATE invoices
    SET customer_id = ${customerId}, amount = ${amountInCents}, status = ${status}
    WHERE id = ${id}
  `;

  revalidatePath("/dashboard/invoices");
  redirect("/dashboard/invoices");
}
```

Mirip dengan action createInvoice, di sini Anda:

- Mengekstrak data dari formData.
- Memvalidasi tipe dengan Zod.
- Mengkonversi jumlah ke sen.
- Meneruskan variabel ke kueri SQL Anda.
- Memanggil revalidatePath untuk menghapus cache klien dan membuat permintaan server baru.
- Memanggil redirect untuk mengarahkan pengguna ke halaman faktur.

Uji coba dengan mengedit faktur. Setelah mengirimkan formulir, Anda harus diarahkan ke halaman faktur, dan faktur harus diperbarui.

### Menghapus faktur

Untuk menghapus faktur menggunakan Server Action, bungkus tombol hapus dalam elemen `<form>` dan pass ID ke Server Action menggunakan bind:

```javascript
// /app/ui/invoices/buttons.js
import { deleteInvoice } from "@/app/lib/actions";

// ...

export function DeleteInvoice({ id }) {
  const deleteInvoiceWithId = deleteInvoice.bind(null, id);

  return (
    <form action={deleteInvoiceWithId}>
      <button type="submit" className="rounded-md border p-2 hover:bg-gray-100">
        <span className="sr-only">Delete</span>
        <TrashIcon className="w-4" />
      </button>
    </form>
  );
}
```

Di dalam file actions.js Anda, buat action baru yang disebut `deleteInvoice`.

```javascript
// /app/lib/actions.js
export async function deleteInvoice(id: string) {
  await sql`DELETE FROM invoices WHERE id = ${id}`;
  revalidatePath("/dashboard/invoices");
}
```

Karena action ini dipanggil di rute `/dashboard/invoices`, Anda tidak perlu memanggil redirect. Memanggil revalidatePath akan memicu permintaan server baru dan merender ulang tabel.

### Bacaan lebih lanjut

Pada bab ini, Anda belajar cara menggunakan Server Actions untuk memutasi data. Anda juga belajar cara menggunakan API revalidatePath untuk merevalidasi cache Next.js dan redirect untuk mengarahkan pengguna ke halaman baru.

Anda juga dapat membaca lebih lanjut tentang keamanan dengan Server Actions untuk pembelajaran tambahan.
