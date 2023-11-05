## Event and Listener

Misal, terdapat sebuah sistem yang dapat melakukan pemesanan atau order. Order pada project Laravel merupakan sebuah model. Salah satu contoh event pada sistem tersebut adalah ketika pemesanan dibuat atau order created. Ketika order created, tugas listener adalah secara otomatis mengepost order tersebut pada API pihak ketiga, yakni vendor dan inventori. Berikut tahapan implementasinya.
- Membuat model Order beserta databasenya dengan command
```
php artisan make:model Order -m
php artisan migrate
```
- Membuat event dengan command 
```
php artisan make:event OrderPlaced
```
- Mengisi parameter constructor App\Events\OrderPlaced sehingga data order dapat disalurkan pada listener dengan kode
```
public function __construct(public Order $order)
    {
        //
    }
```
- Mendeklarasikan event OrderPlaced pada App\Models\Order sehingga dipanggil setiap order dibuat dengan kode
```
protected $dispatchesEvents = [
    'created' => OrderPlaced::class
];
```
- Membuat listener dengan command
```
php artisan make:listener UpdateInventoryAboutOrder --event=OrderPlaced
php artisan make:listener UpdateVendorAboutOrder --event=OrderPlaced
```
- Mengatur App\Listeners\UpdateInventoryAboutOrder agar mengepost order pada API pihak ketiga setiap event terjadi dengan kode
```
public function handle(OrderPlaced $event): void
{
    Http::post('https://inventory.company.com', [
        'order' => $event->order->toArray()
    ]);
}
```
- Mengatur App\Listeners\UpdateVendorAboutOrder agar mengepost order pada API pihak ketiga setiap event terjadi dengan kode
```
public function handle(OrderPlaced $event): void
{
    Http::post('https://vendor.company.com', [
        'order' => $event->order->toArray()
    ]);
}
```
- Namun, karena API tersebut hanyalah permisalan yang belum nyata, agar dapat didemokan, API tersebut digantikan dengan file storage\logs\laravel.log sehingga kode pada App\Listeners\UpdateInventoryAboutOrder dikode menjadi
```
public function handle(OrderPlaced $event): void
{
    info('Inventory was updated about order '. $event->order->id);
}
```
- Begitu juga dengan kode pada App\Listeners\UpdateVendorAboutOrder dikode menjadi
```
public function handle(OrderPlaced $event): void
{
    info('Vendor was updated about order '. $event->order->id);
}
```
- Menggunakan queue pada listener untuk meningkatkan performa waktu sehingga listener tidak berjalan secara simultan dengan implementasikannya pada kelas listener
```
class UpdateInventoryAboutOrder implements ShouldQueue
```
```
class UpdateVendorAboutOrder implements ShouldQueue
```
- Membuat perintah create order pada routes\web sehingga order akan dibuat setiap page direfresh dengan kode
```
Route::get('/', function () {
    Order::create();
    return 'Thank you for your order!';
});
```
- Jika page direfresh dua kali, file log pada storage\logs\laravel.log berisi
```
[2023-11-05 12:49:49] local.INFO: Inventory was updated about order 1  
[2023-11-05 12:49:49] local.INFO: Vendor was updated about order 1  
[2023-11-05 12:50:46] local.INFO: Inventory was updated about order 2  
[2023-11-05 12:50:46] local.INFO: Vendor was updated about order 2
```
<a href="https://www.youtube.com/watch?v=K66ulWMj_O0">Sumber</a>