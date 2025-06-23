بالتأكيد! سأقوم بتصميم مشروع Laravel كامل يلبي متطلباتك، مع الأخذ في الاعتبار التصميمات الأمامية التي تم إنشاؤها مسبقًا، وتضمين نظام الأدوار، وتوليد أكواد QR، ووظائف CRUD.

هذا المشروع سيكون بمثابة هيكل أساسي وقابل للتوسع. سأفترض بعض التفاصيل لجعله كاملاً قدر الإمكان.

**ملاحظات هامة قبل البدء:**

1.  **بيئة التطوير:** ستحتاج إلى بيئة تطوير Laravel (مثل Laragon, XAMPP, Docker مع Laravel Sail, أو Valet).
2.  **الشعار والصور:** يجب أن تضع ملف الشعار (مثلاً `logo.png`) وصور الحيوانات في المسارات المناسبة (مثلاً `public/images/`).
3.  **تصميم الواجهات:** سأدمج تصاميم الـ Bootstrap التي قمنا بها سابقاً في ملفات Blade Views.
4.  **اللغات:** لتسهيل الشرح، سأركز على اللغة العربية في الواجهة الأمامية الأساسية للوحة التحكم و CRUD. صفحة الكلب الشارد التي طلبتها بلغتين ستبقى منفصلة كصفحة عامة.
5.  **الميزات غير المشمولة بشكل كامل في هذا الهيكل المبدئي:**
    *   نظام المقالات والكتب.
    *   المتجر وبوابات الدفع.
    *   بعض التفاصيل الدقيقة للملفات الصحية/الغذائية/الإنتاجية (لكن الهيكل موجود).
    *   إشعارات البريد الإلكتروني.
    *   هذه الميزات تحتاج لتطوير إضافي بعد بناء الهيكل الأساسي.

---

**التعليمات وخطوات التنفيذ:**

**الخطوة 1: إعداد مشروع Laravel**

1.  **إنشاء مشروع Laravel جديد:**
    ```bash
    composer create-project laravel/laravel dr_em_app
    cd dr_em_app
    ```
2.  **تكوين قاعدة البيانات:**
    افتح ملف `.env` وقم بتعديل بيانات قاعدة البيانات الخاصة بك:
    ```dotenv
    DB_CONNECTION=mysql
    DB_HOST=127.0.0.1
    DB_PORT=3306
    DB_DATABASE=dr_em_db # غيّر هذا إلى اسم قاعدتك
    DB_USERNAME=root     # غيّر هذا إلى اسم مستخدم قاعدة بياناتك
    DB_PASSWORD=         # غيّر هذا إلى كلمة مرور قاعدة بياناتك
    ```
3.  **تثبيت Laravel UI للمصادقة:**
    ```bash
    composer require laravel/ui
    php artisan ui bootstrap --auth
    npm install && npm run dev
    ```
    *ملاحظة:* `npm install && npm run dev` مهم لتجميع أصول Bootstrap و JavaScript.

4.  **تثبيت حزمة توليد QR Code:**
    ```bash
    composer require simplesoftwareio/simple-qrcode
    ```

---

**الخطوة 2: إنشاء الهيكل الأساسي لقاعدة البيانات (Migrations)**

سأفترض أن لدينا `User`، `Pet`، وربما جداول أخرى لتفاصيل الحيوان.

1.  **تعديل جدول `users` (لإضافة الأدوار):**
    ```bash
    php artisan make:migration add_role_to_users_table --table=users
    ```
    افتح الملف الجديد في `database/migrations/` (سيكون اسمه مشابهاً لـ `2023_11_20_xxxxxx_add_role_to_users_table.php`) وأضف الكود التالي في دالتي `up` و `down`:

    ```php
    // ...
    public function up(): void
    {
        Schema::table('users', function (Blueprint $table) {
            $table->string('role')->default('regular_user')->after('email'); // admin, data_entry, regular_user
        });
    }

    public function down(): void
    {
        Schema::table('users', function (Blueprint $table) {
            $table->dropColumn('role');
        });
    }
    // ...
    ```

2.  **إنشاء جداول `pets`, `medical_records`, `feeding_records`, `breeding_records`, `media`, `qr_code_links`:**
    ```bash
    php artisan make:model Pet -m
    php artisan make:model MedicalRecord -m
    php artisan make:model FeedingRecord -m
    php artisan make:model BreedingRecord -m
    php artisan make:model Media -m
    php artisan make:model QrCodeLink -m
    ```
    افتح ملفات الـ Migration التي تم إنشاؤها في `database/migrations/` واملأها بالهيكل التالي:

    *   **`create_pets_table.php`**:
        ```php
        // ...
        public function up(): void
        {
            Schema::create('pets', function (Blueprint $table) {
                $table->id();
                $table->string('name');
                $table->string('species'); // cat, dog, bird, etc.
                $table->string('breed')->nullable();
                $table->string('gender')->nullable(); // male, female, unknown
                $table->date('date_of_birth')->nullable();
                $table->string('color')->nullable();
                $table->text('distinguishing_marks')->nullable();
                $table->string('microchip_number')->nullable();
                $table->uuid('uuid')->unique(); // Unique identifier for public links
                $table->foreignId('owner_id')->nullable()->constrained('users')->onDelete('set null'); // nullable for stray/unclaimed
                $table->string('image_path')->nullable(); // Main image for the pet
                $table->timestamps();
            });
        }
        // ...
        ```

    *   **`create_medical_records_table.php`**:
        ```php
        // ...
        public function up(): void
        {
            Schema::create('medical_records', function (Blueprint $table) {
                $table->id();
                $table->foreignId('pet_id')->constrained()->onDelete('cascade');
                $table->date('record_date');
                $table->string('record_type'); // e.g., 'vaccination', 'disease', 'allergy', 'checkup'
                $table->string('vaccine_type')->nullable(); // For vaccination records
                $table->string('allergy_type')->nullable(); // For allergy records
                $table->text('description')->nullable(); // General notes
                $table->string('attachment_path')->nullable(); // Path to vaccine certificate, report, etc.
                $table->timestamps();
            });
        }
        // ...
        ```

    *   **`create_feeding_records_table.php`**:
        ```php
        // ...
        public function up(): void
        {
            Schema::create('feeding_records', function (Blueprint $table) {
                $table->id();
                $table->foreignId('pet_id')->constrained()->onDelete('cascade');
                $table->string('main_food')->nullable();
                $table->integer('meals_per_day')->nullable();
                $table->string('food_quantity')->nullable();
                $table->text('dietary_additions')->nullable();
                $table->text('supplements')->nullable(); // Natural or medicinal
                $table->text('food_allergies')->nullable();
                $table->string('favorite_food')->nullable();
                $table->timestamps();
            });
        }
        // ...
        ```

    *   **`create_breeding_records_table.php`**:
        ```php
        // ...
        public function up(): void
        {
            Schema::create('breeding_records', function (Blueprint $table) {
                $table->id();
                $table->foreignId('pet_id')->constrained()->onDelete('cascade'); // The parent pet involved
                $table->foreignId('mate_id')->nullable()->constrained('pets')->onDelete('set null'); // The other parent
                $table->date('pairing_date')->nullable();
                $table->date('egg_lay_date')->nullable(); // For birds
                $table->date('hatch_date')->nullable();   // For birds
                $table->date('pregnancy_date')->nullable(); // For mammals
                $table->date('birth_date')->nullable();     // For mammals
                $table->integer('num_offspring')->nullable(); // Number of kits/puppies/chicks
                $table->date('weaning_date')->nullable();
                $table->text('notes')->nullable();
                $table->timestamps();
            });
        }
        // ...
        ```

    *   **`create_media_table.php`**:
        ```php
        // ...
        public function up(): void
        {
            Schema::create('media', function (Blueprint $table) {
                $table->id();
                $table->foreignId('pet_id')->constrained()->onDelete('cascade');
                $table->string('file_path');
                $table->string('file_type'); // 'image', 'video'
                $table->text('description')->nullable();
                $table->timestamps();
            });
        }
        // ...
        ```

    *   **`create_qr_code_links_table.php`**: (هذا الجدول يمكن أن يكون اختياريًا إذا كنت تعتمد بالكامل على `pets.uuid` وتخزن مسار الـ QR في جدول `pets` مباشرة). لكن لزيادة المرونة وتتبع كل رمز QR منفصل:
        ```php
        // ...
        public function up(): void
        {
            Schema::create('qr_code_links', function (Blueprint $table) {
                $table->id();
                $table->foreignId('pet_id')->constrained()->onDelete('cascade');
                $table->string('qr_identifier')->unique(); // This will be the UUID from the Pet table or a new unique string
                $table->string('qr_image_path')->nullable(); // Path to the generated QR code image
                $table->timestamps();
            });
        }
        // ...
        ```

3.  **تشغيل الـ Migrations:**
    ```bash
    php artisan migrate
    ```

---

**الخطوة 3: تعريف الـ Models والعلاقات**

افتح الملفات في `app/Models/` واملأها بالعلاقات والـ fillable properties.

*   **`app/Models/User.php`**:
    ```php
    <?php

    namespace App\Models;

    use Illuminate\Contracts\Auth\MustVerifyEmail;
    use Illuminate\Database\Eloquent\Factories\HasFactory;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;
    use Laravel\Sanctum\HasApiTokens;

    class User extends Authenticatable
    {
        use HasApiTokens, HasFactory, Notifiable;

        protected $fillable = [
            'name',
            'email',
            'password',
            'role', // أضفنا هذا
        ];

        protected $hidden = [
            'password',
            'remember_token',
        ];

        protected $casts = [
            'email_verified_at' => 'datetime',
        ];

        // تعريف الأدوار
        public function isAdmin()
        {
            return $this->role === 'admin';
        }

        public function isDataEntry()
        {
            return $this->role === 'data_entry';
        }

        public function isRegularUser()
        {
            return $this->role === 'regular_user';
        }

        public function pets()
        {
            return $this->hasMany(Pet::class, 'owner_id');
        }
    }
    ```

*   **`app/Models/Pet.php`**:
    ```php
    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Factories\HasFactory;
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Support\Str; // لاستخدام UUID

    class Pet extends Model
    {
        use HasFactory;

        protected $fillable = [
            'name', 'species', 'breed', 'gender', 'date_of_birth', 'color',
            'distinguishing_marks', 'microchip_number', 'uuid', 'owner_id', 'image_path'
        ];

        protected static function boot()
        {
            parent::boot();
            static::creating(function ($pet) {
                if (empty($pet->uuid)) {
                    $pet->uuid = (string) Str::uuid();
                }
            });
        }

        public function owner()
        {
            return $this->belongsTo(User::class, 'owner_id');
        }

        public function medicalRecords()
        {
            return $this->hasMany(MedicalRecord::class);
        }

        public function feedingRecords()
        {
            return $this->hasOne(FeedingRecord::class); // غالباً سجل غذائي واحد لكل حيوان
        }

        public function breedingRecords()
        {
            return $this->hasMany(BreedingRecord::class);
        }

        public function media()
        {
            return $this->hasMany(Media::class);
        }

        public function qrCodeLink()
        {
            return $this->hasOne(QrCodeLink::class);
        }
    }
    ```

*   **`app/Models/MedicalRecord.php`**:
    ```php
    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Factories\HasFactory;
    use Illuminate\Database\Eloquent\Model;

    class MedicalRecord extends Model
    {
        use HasFactory;

        protected $fillable = [
            'pet_id', 'record_date', 'record_type', 'vaccine_type',
            'allergy_type', 'description', 'attachment_path'
        ];

        public function pet()
        {
            return $this->belongsTo(Pet::class);
        }
    }
    ```

*   **`app/Models/FeedingRecord.php`**:
    ```php
    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Factories\HasFactory;
    use Illuminate\Database\Eloquent\Model;

    class FeedingRecord extends Model
    {
        use HasFactory;

        protected $fillable = [
            'pet_id', 'main_food', 'meals_per_day', 'food_quantity',
            'dietary_additions', 'supplements', 'food_allergies', 'favorite_food'
        ];

        public function pet()
        {
            return $this->belongsTo(Pet::class);
        }
    }
    ```

*   **`app/Models/BreedingRecord.php`**:
    ```php
    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Factories\HasFactory;
    use Illuminate\Database\Eloquent\Model;

    class BreedingRecord extends Model
    {
        use HasFactory;

        protected $fillable = [
            'pet_id', 'mate_id', 'pairing_date', 'egg_lay_date', 'hatch_date',
            'pregnancy_date', 'birth_date', 'num_offspring', 'weaning_date', 'notes'
        ];

        public function pet()
        {
            return $this->belongsTo(Pet::class, 'pet_id');
        }

        public function mate()
        {
            return $this->belongsTo(Pet::class, 'mate_id');
        }
    }
    ```

*   **`app/Models/Media.php`**:
    ```php
    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Factories\HasFactory;
    use Illuminate\Database\Eloquent\Model;

    class Media extends Model
    {
        use HasFactory;

        protected $fillable = [
            'pet_id', 'file_path', 'file_type', 'description'
        ];

        public function pet()
        {
            return $this->belongsTo(Pet::class);
        }
    }
    ```

*   **`app/Models/QrCodeLink.php`**:
    ```php
    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Factories\HasFactory;
    use Illuminate\Database\Eloquent\Model;

    class QrCodeLink extends Model
    {
        use HasFactory;

        protected $fillable = [
            'pet_id', 'qr_identifier', 'qr_image_path'
        ];

        public function pet()
        {
            return $this->belongsTo(Pet::class);
        }
    }
    ```

---

**الخطوة 4: Middleware للأدوار والصلاحيات**

1.  **إنشاء Middleware:**
    ```bash
    php artisan make:middleware CheckUserRole
    ```
    افتح `app/Http/Middleware/CheckUserRole.php` واملأه بالكود التالي:

    ```php
    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Auth;
    use Symfony\Component\HttpFoundation\Response;

    class CheckUserRole
    {
        public function handle(Request $request, Closure $next, ...$roles): Response
        {
            if (!Auth::check()) {
                return redirect('/login'); // أو أي صفحة دخول
            }

            $user = Auth::user();

            foreach ($roles as $role) {
                if ($user->role === $role) {
                    return $next($request);
                }
            }

            // إذا لم يكن لديه أي من الأدوار المطلوبة
            abort(403, 'Unauthorized action.'); // صفحة خطأ 403 (غير مصرح لك)
        }
    }
    ```

2.  **تسجيل الـ Middleware:**
    افتح `app/Http/Kernel.php` وأضف الـ Middleware الجديد إلى `$routeMiddleware` array:

    ```php
    // ...
    protected $routeMiddleware = [
        // ...
        'role' => \App\Http\Middleware\CheckUserRole::class,
    ];
    // ...
    ```

---

**الخطوة 5: الـ Seeders (لإنشاء مستخدمين افتراضيين)**

1.  **إنشاء Seeder:**
    ```bash
    php artisan make:seeder UserSeeder
    ```
    افتح `database/seeders/UserSeeder.php` واملأه بالكود التالي:

    ```php
    <?php

    namespace Database\Seeders;

    use Illuminate\Database\Console\Seeds\WithoutModelEvents;
    use Illuminate\Database\Seeder;
    use App\Models\User;
    use Illuminate\Support\Facades\Hash;

    class UserSeeder extends Seeder
    {
        public function run(): void
        {
            User::create([
                'name' => 'Admin User',
                'email' => 'admin@example.com',
                'password' => Hash::make('password'),
                'role' => 'admin',
            ]);

            User::create([
                'name' => 'Data Entry User',
                'email' => 'dataentry@example.com',
                'password' => Hash::make('password'),
                'role' => 'data_entry',
            ]);

            User::create([
                'name' => 'Regular User',
                'email' => 'user@example.com',
                'password' => Hash::make('password'),
                'role' => 'regular_user',
            ]);
        }
    }
    ```

2.  **تشغيل الـ Seeder:**
    افتح `database/seeders/DatabaseSeeder.php` وأضف `UserSeeder::class` إلى دالة `run`:

    ```php
    // ...
    public function run(): void
    {
        $this->call(UserSeeder::class);
    }
    // ...
    ```
    ثم قم بتشغيل الـ Seeder:
    ```bash
    php artisan db:seed
    ```

---

**الخطوة 6: الـ Controllers**

1.  **`app/Http/Controllers/HomeController.php`**: (لوحة التحكم)
    ```php
    <?php

    namespace App\Http\Controllers;

    use App\Models\Pet;
    use Illuminate\Http\Request;

    class HomeController extends Controller
    {
        public function __construct()
        {
            $this->middleware('auth');
        }

        public function index()
        {
            $user = auth()->user();
            if ($user->isAdmin() || $user->isDataEntry()) {
                $totalPets = Pet::count();
                // يمكنك إضافة منطق للمهام والتنبيهات
                $upcomingTasks = 2; // Dummy data
                $healthAlerts = 1; // Dummy data
                return view('home', compact('totalPets', 'upcomingTasks', 'healthAlerts'));
            } else {
                // للمستخدم العادي، عرض حيواناته فقط أو لوحة تحكم أبسط
                $totalPets = Pet::where('owner_id', $user->id)->count();
                return view('home', compact('totalPets'));
            }
        }
    }
    ```

2.  **`app/Http/Controllers/PetController.php`**: (إدارة الحيوانات)
    ```php
    <?php

    namespace App\Http\Controllers;

    use App\Models\Pet;
    use App\Models\FeedingRecord;
    use App\Models\MedicalRecord;
    use App\Models\BreedingRecord;
    use App\Models\Media;
    use App\Models\QrCodeLink;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Storage;
    use SimpleSoftwareIO\QrCode\Facades\QrCode;

    class PetController extends Controller
    {
        public function __construct()
        {
            // حماية جميع الدوال إلا showPublic
            $this->middleware('auth')->except('showPublic');
            // حماية دوال معينة بأدوار محددة
            $this->middleware('role:admin,data_entry')->only(['create', 'store', 'edit', 'update', 'destroy', 'generateQRCodes', 'showQRGenerationForm']);
            $this->middleware('role:admin,data_entry,regular_user')->only(['index', 'show']);
        }

        // عرض قائمة الحيوانات
        public function index()
        {
            $user = auth()->user();
            if ($user->isAdmin() || $user->isDataEntry()) {
                $pets = Pet::all();
            } else {
                $pets = $user->pets; // للمستخدم العادي، يعرض حيواناته فقط
            }
            return view('pets.index', compact('pets'));
        }

        // عرض نموذج إضافة حيوان جديد
        public function create()
        {
            return view('pets.create');
        }

        // حفظ حيوان جديد
        public function store(Request $request)
        {
            $request->validate([
                'name' => 'required|string|max:255',
                'species' => 'required|string|max:255',
                'breed' => 'nullable|string|max:255',
                'gender' => 'nullable|string|max:255',
                'date_of_birth' => 'nullable|date',
                'color' => 'nullable|string|max:255',
                'image' => 'nullable|image|mimes:jpeg,png,jpg,gif,svg|max:2048',
                // Add validation for other fields
            ]);

            $pet = new Pet($request->except('image'));
            $pet->owner_id = auth()->id(); // أو يمكنك تحديد المالك من فورمة الأدمن

            if ($request->hasFile('image')) {
                $imagePath = $request->file('image')->store('pets_images', 'public');
                $pet->image_path = $imagePath;
            }

            $pet->save();

            // إنشاء سجل QR Code مبدئي للحيوان
            $this->generateSingleQRCode($pet);

            return redirect()->route('pets.index')->with('success', 'تم إضافة الحيوان بنجاح!');
        }

        // عرض تفاصيل حيوان (للمالك أو مدخل البيانات/المدير)
        public function show(Pet $pet)
        {
            // تحقق من أن المستخدم الحالي هو المالك أو أدمن/مدخل بيانات
            if (auth()->user()->id !== $pet->owner_id && !auth()->user()->isAdmin() && !auth()->user()->isDataEntry()) {
                abort(403, 'غير مصرح لك بالوصول إلى هذا الملف.');
            }
            // تحميل العلاقات اللازمة
            $pet->load(['medicalRecords', 'feedingRecords', 'breedingRecords', 'media', 'qrCodeLink']);
            return view('pets.show', compact('pet'));
        }

        // عرض نموذج تعديل حيوان
        public function edit(Pet $pet)
        {
             // تحقق من أن المستخدم الحالي هو المالك أو أدمن/مدخل بيانات
            if (auth()->user()->id !== $pet->owner_id && !auth()->user()->isAdmin() && !auth()->user()->isDataEntry()) {
                abort(403, 'غير مصرح لك بتعديل هذا الملف.');
            }
            return view('pets.edit', compact('pet'));
        }

        // تحديث بيانات حيوان
        public function update(Request $request, Pet $pet)
        {
             // تحقق من أن المستخدم الحالي هو المالك أو أدمن/مدخل بيانات
            if (auth()->user()->id !== $pet->owner_id && !auth()->user()->isAdmin() && !auth()->user()->isDataEntry()) {
                abort(403, 'غير مصرح لك بتعديل هذا الملف.');
            }

            $request->validate([
                'name' => 'required|string|max:255',
                'species' => 'required|string|max:255',
                // Add validation for other fields
            ]);

            $pet->update($request->except('image'));

            if ($request->hasFile('image')) {
                // حذف الصورة القديمة إذا وجدت
                if ($pet->image_path) {
                    Storage::disk('public')->delete($pet->image_path);
                }
                $imagePath = $request->file('image')->store('pets_images', 'public');
                $pet->image_path = $imagePath;
                $pet->save();
            }

            // تحديث أو إنشاء سجل التغذية
            if ($request->has('main_food') || $request->has('meals_per_day')) { // إذا كان هناك أي حقل متعلق بالتغذية
                $pet->feedingRecords()->updateOrCreate(
                    ['pet_id' => $pet->id],
                    $request->only(['main_food', 'meals_per_day', 'food_quantity', 'dietary_additions', 'supplements', 'food_allergies', 'favorite_food'])
                );
            }

            return redirect()->route('pets.show', $pet->id)->with('success', 'تم تحديث بيانات الحيوان بنجاح!');
        }

        // حذف حيوان
        public function destroy(Pet $pet)
        {
            // فقط المدير أو مدخل البيانات يمكنه الحذف
            if (!auth()->user()->isAdmin() && !auth()->user()->isDataEntry()) {
                abort(403, 'غير مصرح لك بحذف هذا الحيوان.');
            }

            // حذف صورة الحيوان والـ QR Code إذا وجدت
            if ($pet->image_path) {
                Storage::disk('public')->delete($pet->image_path);
            }
            if ($pet->qrCodeLink && $pet->qrCodeLink->qr_image_path) {
                Storage::disk('public')->delete($pet->qrCodeLink->qr_image_path);
            }

            $pet->delete();
            return redirect()->route('pets.index')->with('success', 'تم حذف الحيوان بنجاح.');
        }

        // عرض صفحة الحيوان للعامة (عبر QR Code)
        public function showPublic($uuid)
        {
            $pet = Pet::where('uuid', $uuid)->firstOrFail();

            // تحميل العلاقات المطلوبة للعرض العام فقط
            $pet->load(['medicalRecords', 'feedingRecords', 'media']);

            // لا يوجد صلاحيات تعديل هنا، فقط عرض
            return view('pets.public_profile', compact('pet'));
        }

        // عرض نموذج توليد أكواد QR
        public function showQRGenerationForm()
        {
            $pets = Pet::all(); // يمكن فلترة الحيوانات التي ليس لديها QR Code
            return view('qrcodes.generate', compact('pets'));
        }

        // توليد مجموعة من أكواد QR
        public function generateQRCodes(Request $request)
        {
            $request->validate([
                'num_qrcodes' => 'required_without_all:selected_pets|integer|min:1|max:500',
                'selected_pets' => 'required_without_all:num_qrcodes|array',
            ]);

            $generatedQRs = [];
            $petsToGenerateFor = [];

            if ($request->has('selected_pets')) {
                $petsToGenerateFor = Pet::whereIn('id', $request->selected_pets)->get();
            } elseif ($request->num_qrcodes) {
                // إذا تم طلب عدد معين، يمكن إنشاء حيوانات وهمية أو اختيار حيوانات بدون QR
                // هنا نفترض إنشاء حيوانات جديدة لأغراض التجربة
                for ($i = 0; $i < $request->num_qrcodes; $i++) {
                    $pet = Pet::create([
                        'name' => 'Unnamed Pet ' . ($i + 1),
                        'species' => 'Unknown',
                        'gender' => 'Unknown',
                        'uuid' => (string) Str::uuid(), // تأكد من توليد UUID جديد
                        'owner_id' => auth()->user()->isAdmin() ? null : auth()->id(), // إذا كان أدمن يمكن أن يكون بلا مالك
                    ]);
                    $petsToGenerateFor[] = $pet;
                }
            }

            foreach ($petsToGenerateFor as $pet) {
                if (!$pet->qrCodeLink) { // توليد فقط إذا لم يكن لديه QR Code بالفعل
                    $qrCodeImage = QrCode::format('png')
                                    ->size(300)
                                    ->generate(route('pets.show.public', ['uuid' => $pet->uuid]));

                    $qrImagePath = 'qrcodes/' . $pet->uuid . '.png';
                    Storage::disk('public')->put($qrImagePath, $qrCodeImage);

                    $pet->qrCodeLink()->create([
                        'qr_identifier' => $pet->uuid,
                        'qr_image_path' => $qrImagePath,
                    ]);
                    $generatedQRs[] = ['pet' => $pet, 'qr_path' => Storage::url($qrImagePath)];
                }
            }

            return view('qrcodes.index', compact('generatedQRs'))->with('success', 'تم توليد رموز QR بنجاح!');
        }

        // دالة مساعدة لتوليد QR Code واحد عند إنشاء الحيوان
        private function generateSingleQRCode(Pet $pet)
        {
            // تأكد أن الـ UUID موجود قبل التوليد
            if (empty($pet->uuid)) {
                $pet->uuid = (string) Str::uuid();
                $pet->save();
            }

            $qrCodeImage = QrCode::format('png')
                            ->size(300)
                            ->generate(route('pets.show.public', ['uuid' => $pet->uuid]));

            $qrImagePath = 'qrcodes/' . $pet->uuid . '.png';
            Storage::disk('public')->put($qrImagePath, $qrCodeImage);

            $pet->qrCodeLink()->updateOrCreate(
                ['pet_id' => $pet->id],
                ['qr_identifier' => $pet->uuid, 'qr_image_path' => $qrImagePath]
            );
        }
    }
    ```

---

**الخطوة 7: الـ Routes (في `routes/web.php`)**

```php
<?php

use Illuminate\Support\Facades\Route;
use Illuminate\Support\Facades\Auth;
use App\Http\Controllers\HomeController;
use App\Http\Controllers\PetController;

/*
|--------------------------------------------------------------------------
| Web Routes
|--------------------------------------------------------------------------
|
| Here is where you can register web routes for your application. These
| routes are loaded by the RouteServiceProvider and all of them will
| be assigned to the "web" middleware group. Make something great!
|
*/

// صفحة الهبوط الافتراضية
Route::get('/', function () {
    return view('welcome');
});

// مسارات المصادقة (Auth Routes)
Auth::routes();

// مسارات لوحة التحكم (Dashboard) - تتطلب مصادقة
Route::get('/home', [HomeController::class, 'index'])->name('home');

// مسار عرض الحيوان للعامة (عبر QR Code) - لا يتطلب مصادقة
Route::get('/public/pets/{uuid}', [PetController::class, 'showPublic'])->name('pets.show.public');

// مسارات إدارة الحيوانات (Pet Management) - تتطلب مصادقة ودور محدد
Route::middleware(['auth', 'role:admin,data_entry,regular_user'])->group(function () {
    Route::resource('pets', PetController::class);

    // إضافة مسارات فرعية للملفات التفصيلية (يمكن دمجها في PetController@update)
    // Route::post('/pets/{pet}/medical-record', [PetController::class, 'storeMedicalRecord'])->name('pets.medical_records.store');
    // Route::post('/pets/{pet}/feeding-record', [PetController::class, 'storeFeedingRecord'])->name('pets.feeding_records.store');
    // Route::post('/pets/{pet}/breeding-record', [PetController::class, 'storeBreedingRecord'])->name('pets.breeding_records.store');
    // Route::post('/pets/{pet}/media', [PetController::class, 'storeMedia'])->name('pets.media.store');

    // مسارات المهام اليومية (Tasks)
    Route::get('/tasks', function () { return view('tasks'); })->name('tasks');

    // مسارات التنبيهات الصحية (Health Alerts)
    Route::get('/health-alerts', function () { return view('health_alerts'); })->name('health-alerts');

    // مسارات توليد QR Code - تتطلب صلاحيات المدير/مدخل البيانات
    Route::middleware(['role:admin,data_entry'])->group(function () {
        Route::get('/qrcodes/generate', [PetController::class, 'showQRGenerationForm'])->name('qrcodes.generate.form');
        Route::post('/qrcodes/generate', [PetController::class, 'generateQRCodes'])->name('qrcodes.generate.submit');
        Route::get('/qrcodes', [PetController::class, 'indexQRCodes'])->name('qrcodes.index'); // لعرض قائمة الـ QRs
    });
});

// يمكنك إضافة مسارات أخرى هنا (مثل المحتوى التعليمي، المتجر)
```

---

**الخطوة 8: الـ Views (ملفات Blade)**

**إنشاء المجلدات أولاً:**
```bash
mkdir -p resources/views/layouts
mkdir -p resources/views/auth
mkdir -p resources/views/pets
mkdir -p resources/views/qrcodes
```

1.  **`resources/views/layouts/app.blade.php`**: (القالب الرئيسي - ادمج Navbar و Footer من التصميمات السابقة)
    ```blade
    <!DOCTYPE html>
    <html lang="ar" dir="rtl">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <meta name="csrf-token" content="{{ csrf_token() }}">
        <title>Dr.Em - @yield('title', 'إدارة الحيوانات الأليفة')</title>

        <!-- Fonts -->
        <link rel="dns-prefetch" href="//fonts.gstatic.com">
        <link href="https://fonts.googleapis.com/css?family=Nunito" rel="stylesheet">
        <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;500;700&display=swap" rel="stylesheet">

        <!-- Bootstrap CSS -->
        <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-T3c6CoIi6uLrA9TneNEoa7RxnatzjcDSCmG1MXxSR1GAsXEV/Dwwykc2MPK8M2HN" crossorigin="anonymous">
        <!-- Font Awesome -->
        <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css" integrity="sha512-DTOQO9RWCH3ppGqcWaEA1BIZOC6xxalwEsw9c2QQeAIftl+Vegovlnee1c9QX4TctnWMn13TZye+giMm8e2LwA==" crossorigin="anonymous" referrerpolicy="no-referrer" />

        <!-- Custom CSS (من ملف style.css السابق) -->
        <style>
            body { background-color: #f8f9fa; font-family: 'Tajawal', sans-serif; }
            .navbar-brand img { max-height: 40px; margin-right: 10px; }
            .dashboard-card { transition: transform 0.2s ease-in-out; }
            .dashboard-card:hover { transform: translateY(-5px); box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1); }
            .pet-card img { height: 200px; object-fit: cover; }
            .pet-profile-img { max-width: 100%; height: auto; max-height: 400px; border-radius: 0.5rem; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1); object-fit: cover;}
            .profile-details dt { font-weight: 600; color: #555; }
            .profile-details dd { margin-bottom: 0.8rem; color: #333; }
            .task-item { border-left: 4px solid #0d6efd; margin-bottom: 1rem; padding: 1rem; background-color: #fff; border-radius: 0.25rem;}
            .task-item.completed { border-left-color: #198754; text-decoration: line-through; opacity: 0.7;}
            /* Custom Colors */
            .navbar-dark.bg-primary { background-color: #2c3e50 !important; } /* كحلي غامق */
            .btn-primary { background-color: #1abc9c; border-color: #1abc9c; } /* تركواز */
            .btn-primary:hover { background-color: #16a085; border-color: #16a085; }
            .nav-pills .nav-link.active, .nav-pills .show>.nav-link { background-color: #1abc9c; } /* تركواز للتابات النشطة */
            .nav-link { color: #2c3e50; }
            .nav-link:hover { color: #1abc9c; }
            /* Specific for pet profiles */
            .cat-profile-img { max-width: 200px; height: 200px; object-fit: cover; border-radius: 50%; border: 5px solid #e9ecef; margin: 0 auto; display: block; box-shadow: 0 4px 10px rgba(0, 0, 0, 0.15);}
            .bg-cat-header { background: linear-gradient(135deg, #c3cfe2 0%, #f5f7fa 100%);}
            .cat-profile .nav-pills .nav-link.active, .cat-profile .nav-pills .show > .nav-link { background-color: #9b59b6; color: white;}
            .cat-profile .nav-link { color: #9b59b6;} .cat-profile .nav-link:hover { color: #8e44ad;}
            .bird-profile-header { background-color: #f0fdf4; border-bottom: 3px solid #2ecc71; padding: 1.5rem; border-radius: 0.5rem 0.5rem 0 0;}
            .bird-profile-img { max-width: 150px; height: 150px; object-fit: cover; border-radius: 0.375rem; border: 3px solid #fff; box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);}
            .bird-profile .accordion-button { font-weight: 600; color: #1a5d3b;}
            .bird-profile .accordion-button:not(.collapsed) { background-color: #d1fae5; color: #064e3b; box-shadow: inset 0 -1px 0 rgba(0,0,0,.125);}
            .bird-profile .accordion-button:focus { box-shadow: 0 0 0 0.25rem rgba(46, 204, 113, 0.25);}
            .bird-profile .accordion-icon { color: #2ecc71; margin-right: 0.5rem;}
            /* Health Alerts specific */
            .alert-card { border-right-width: 5px; border-left-width: 0; border-top-width: 0; border-bottom-width: 0; border-style: solid; transition: box-shadow 0.3s ease-in-out;}
            .alert-card:hover { box-shadow: 0 .5rem 1rem rgba(0,0,0,.1)!important;}
            .alert-card .pet-icon { font-size: 1.5rem; margin-left: 10px; opacity: 0.7;}
            .alert-card .card-title { font-weight: 600; display: flex; align-items: center;}
            .alert-card .card-title i { font-size: 1.2rem; margin-left: 8px;}
            .alert-card .card-footer { background-color: #f8f9fa; padding-top: 0.5rem; padding-bottom: 0.5rem;}
            .filter-section { background-color: #ffffff; padding: 1rem; border-radius: 0.375rem; margin-bottom: 1.5rem; box-shadow: 0 2px 4px rgba(0,0,0,.05);}

        </style>
    </head>
    <body>
        <div id="app">
            <nav class="navbar navbar-expand-md navbar-dark bg-primary shadow-sm">
                <div class="container-fluid">
                    <a class="navbar-brand fw-bold" href="{{ url('/home') }}">
                        <i class="fas fa-paw me-2"></i>Dr.Em
                    </a>
                    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav" aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
                        <span class="navbar-toggler-icon"></span>
                    </button>
                    <div class="collapse navbar-collapse" id="navbarNav">
                        <ul class="navbar-nav me-auto mb-2 mb-lg-0">
                            <li class="nav-item">
                                <a class="nav-link {{ Request::is('home') ? 'active' : '' }}" href="{{ route('home') }}"><i class="fas fa-tachometer-alt me-1"></i>لوحة التحكم</a>
                            </li>
                            <li class="nav-item">
                                <a class="nav-link {{ Request::is('pets*') ? 'active' : '' }}" href="{{ route('pets.index') }}"><i class="fas fa-dog me-1"></i>حيواناتي</a>
                            </li>
                            <li class="nav-item">
                                <a class="nav-link {{ Request::is('tasks') ? 'active' : '' }}" href="{{ route('tasks') }}"><i class="fas fa-tasks me-1"></i>المهام اليومية</a>
                            </li>
                            <li class="nav-item">
                                <a class="nav-link {{ Request::is('health-alerts') ? 'active' : '' }}" href="{{ route('health-alerts') }}"><i class="fas fa-heartbeat me-1"></i>التنبيهات الصحية</a>
                            </li>
                            @if(Auth::check() && (Auth::user()->isAdmin() || Auth::user()->isDataEntry()))
                            <li class="nav-item">
                                <a class="nav-link {{ Request::is('qrcodes*') ? 'active' : '' }}" href="{{ route('qrcodes.generate.form') }}"><i class="fas fa-qrcode me-1"></i>إدارة الباركود</a>
                            </li>
                            @endif
                            <!-- يمكنك إضافة روابط أخرى هنا مثل المتجر والمقالات -->
                        </ul>

                        <ul class="navbar-nav ms-auto">
                            @guest
                                @if (Route::has('login'))
                                    <li class="nav-item">
                                        <a class="nav-link" href="{{ route('login') }}">{{ __('Login') }}</a>
                                    </li>
                                @endif

                                @if (Route::has('register'))
                                    <li class="nav-item">
                                        <a class="nav-link" href="{{ route('register') }}">{{ __('Register') }}</a>
                                    </li>
                                @endif
                            @else
                                <li class="nav-item dropdown">
                                    <a id="navbarDropdown" class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown" aria-haspopup="true" aria-expanded="false" v-pre>
                                        <i class="fas fa-user-circle me-1"></i>{{ Auth::user()->name }} ({{ Auth::user()->role }})
                                    </a>

                                    <div class="dropdown-menu dropdown-menu-end" aria-labelledby="navbarDropdown">
                                        <a class="dropdown-item" href="#">
                                            <i class="fas fa-user-edit me-2"></i>تعديل الملف الشخصي
                                        </a>
                                        <a class="dropdown-item" href="#">
                                            <i class="fas fa-cog me-2"></i>الإعدادات
                                        </a>
                                        <hr class="dropdown-divider">
                                        <a class="dropdown-item text-danger" href="{{ route('logout') }}"
                                           onclick="event.preventDefault();
                                                         document.getElementById('logout-form').submit();">
                                            <i class="fas fa-sign-out-alt me-2"></i>تسجيل الخروج
                                        </a>

                                        <form id="logout-form" action="{{ route('logout') }}" method="POST" class="d-none">
                                            @csrf
                                        </form>
                                    </div>
                                </li>
                            @endguest
                        </ul>
                    </div>
                </div>
            </nav>

            <main class="py-4">
                @if (session('success'))
                    <div class="container">
                        <div class="alert alert-success alert-dismissible fade show" role="alert">
                            {{ session('success') }}
                            <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
                        </div>
                    </div>
                @endif
                @if (session('error'))
                    <div class="container">
                        <div class="alert alert-danger alert-dismissible fade show" role="alert">
                            {{ session('error') }}
                            <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
                        </div>
                    </div>
                @endif
                @yield('content')
            </main>

            <footer class="bg-light text-center text-lg-start mt-5">
                <div class="text-center p-3" style="background-color: rgba(0, 0, 0, 0.05);">
                    © {{ date('Y') }} Dr.Em - جميع الحقوق محفوظة
                </div>
            </footer>
        </div>

        <!-- Bootstrap JS Bundle -->
        <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js" integrity="sha384-C6RzsynM9kWDrMNeT87bh95OGNyZPhcTNXj1NW7RuBCsyN/o0jlpcV8Qyq46cDfL" crossorigin="anonymous"></script>
        @stack('scripts')
    </body>
    </html>
    ```

2.  **`resources/views/home.blade.php`**: (لوحة التحكم)
    ```blade
    @extends('layouts.app')

    @section('content')
    <div class="container mt-4">
        <div class="row mb-4">
            <div class="col-12">
                <h1 class="display-6 fw-light">نظرة عامة</h1>
                <hr>
            </div>
        </div>

        <div class="row g-4">
            <!-- Card: إجمالي الحيوانات -->
            <div class="col-md-6 col-lg-3">
                <div class="card text-center dashboard-card shadow-sm h-100">
                    <div class="card-body">
                        <i class="fas fa-paw fa-3x text-primary mb-3"></i>
                        <h5 class="card-title">إجمالي الحيوانات</h5>
                        <p class="display-4 fw-bold">{{ $totalPets ?? '0' }}</p>
                        <a href="{{ route('pets.index') }}" class="btn btn-sm btn-outline-primary">إدارة الحيوانات</a>
                    </div>
                </div>
            </div>

            <!-- Card: المهام القادمة -->
            <div class="col-md-6 col-lg-3">
                <div class="card text-center dashboard-card shadow-sm h-100">
                    <div class="card-body">
                        <i class="fas fa-calendar-check fa-3x text-success mb-3"></i>
                        <h5 class="card-title">المهام القادمة (اليوم)</h5>
                        <p class="display-4 fw-bold">{{ $upcomingTasks ?? '0' }}</p>
                        <a href="{{ route('tasks') }}" class="btn btn-sm btn-outline-success">عرض المهام</a>
                    </div>
                </div>
            </div>

            <!-- Card: تنبيهات صحية -->
             <div class="col-md-6 col-lg-3">
                <div class="card text-center dashboard-card shadow-sm h-100">
                    <div class="card-body">
                        <i class="fas fa-heartbeat fa-3x text-danger mb-3"></i>
                        <h5 class="card-title">تنبيهات صحية</h5>
                        <p class="display-4 fw-bold">{{ $healthAlerts ?? '0' }}</p>
                        <small class="text-muted d-block mb-2">(موعد تطعيم قريب)</small>
                        <a href="{{ route('health-alerts') }}" class="btn btn-sm btn-outline-danger">عرض التنبيهات</a>
                    </div>
                </div>
            </div>

            <!-- Card: إضافة حيوان جديد -->
            <div class="col-md-6 col-lg-3">
                 <div class="card text-center dashboard-card shadow-sm h-100 bg-light">
                    <div class="card-body d-flex flex-column justify-content-center">
                        <i class="fas fa-plus-circle fa-3x text-secondary mb-3"></i>
                        <h5 class="card-title">حيوان جديد</h5>
                        <p class="text-muted">إضافة حيوان أليف أو طائر إلى ملفك</p>
                        <a href="{{ route('pets.create') }}" class="btn btn-secondary mt-auto">
                           <i class="fas fa-plus me-1"></i> إضافة الآن
                        </a>
                    </div>
                </div>
            </div>
        </div>

         <!-- قسم إضافي: آخر النشاطات أو المقالات المميزة -->
         <div class="row mt-5">
            <div class="col-12">
                <h2 class="fw-light">آخر التحديثات</h2>
                 <hr>
            </div>
             <div class="col-md-6">
                 <h5><i class="fas fa-newspaper me-2 text-primary"></i>مقالات مميزة</h5>
                 <ul class="list-group list-group-flush">
                    <li class="list-group-item"><a href="#" class="text-decoration-none">أهمية التطعيمات للقطط والكلاب</a></li>
                    <li class="list-group-item"><a href="#" class="text-decoration-none">كيف تختار الغذاء المناسب لطائرك؟</a></li>
                    <li class="list-group-item"><a href="#" class="text-decoration-none">علامات تدل على سعادة حيوانك الأليف</a></li>
                 </ul>
             </div>
             <div class="col-md-6">
                 <h5><i class="fas fa-history me-2 text-primary"></i>آخر الإضافات</h5>
                 <ul class="list-group list-group-flush">
                    <li class="list-group-item">تم إضافة سجل تطعيم لـ "بادي".</li>
                    <li class="list-group-item">تم تحديث المهام اليومية لـ "مشمش".</li>
                    <li class="list-group-item">تم إضافة "لوسي" (قطة) إلى قائمة حيواناتك.</li>
                </ul>
             </div>
         </div>

    </div>
    @endsection
    ```

3.  **`resources/views/pets/index.blade.php`**: (قائمة الحيوانات)
    ```blade
    @extends('layouts.app')

    @section('content')
    <div class="container mt-4">
        <div class="d-flex justify-content-between align-items-center mb-4">
            <h1 class="display-6 fw-light">قائمة حيواناتي الأليفة</h1>
            <div>
                 <a href="{{ route('pets.create') }}" class="btn btn-primary me-2"><i class="fas fa-plus me-1"></i> إضافة حيوان</a>
                 <div class="btn-group" role="group" aria-label="View options">
                    <button type="button" class="btn btn-outline-secondary active"><i class="fas fa-th-large"></i></button>
                    <button type="button" class="btn btn-outline-secondary"><i class="fas fa-list"></i></button>
                 </div>
            </div>
        </div>
        <hr>

        <div class="row g-4">
            @forelse ($pets as $pet)
            <div class="col-md-6 col-lg-4">
                <div class="card pet-card shadow-sm h-100">
                    <img src="{{ $pet->image_path ? asset('storage/' . $pet->image_path) : 'https://via.placeholder.com/200?text=Pet+Image' }}" class="card-img-top" alt="صورة {{ $pet->name }}">
                    <div class="card-body d-flex flex-column">
                        <h5 class="card-title fw-bold">{{ $pet->name }} <i class="fas fa-{{ $pet->species == 'dog' ? 'dog' : ($pet->species == 'cat' ? 'cat' : 'dove') }} ms-1 text-muted"></i></h5>
                        <p class="card-text text-muted">{{ $pet->species }} - {{ $pet->breed ?? 'غير محدد' }}</p>
                        @if($pet->medicalRecords->isNotEmpty())
                         <p class="card-text small">آخر تحديث صحي: {{ $pet->medicalRecords->sortByDesc('record_date')->first()->record_type ?? 'لا يوجد' }} ({{ $pet->medicalRecords->sortByDesc('record_date')->first()->record_date->format('d/m/Y') ?? '' }})</p>
                        @else
                         <p class="card-text small">لا توجد سجلات صحية بعد.</p>
                        @endif
                        <div class="mt-auto text-center">
                            <a href="{{ route('pets.show', $pet->id) }}" class="btn btn-primary w-100"><i class="fas fa-eye me-1"></i> عرض الملف الشخصي</a>
                        </div>
                    </div>
                    <div class="card-footer text-muted small">
                         الرقم التسلسلي: {{ $pet->uuid }}
                         @if($pet->qrCodeLink)
                         <a href="{{ asset('storage/' . $pet->qrCodeLink->qr_image_path) }}" download class="btn btn-link btn-sm p-0 ms-2" title="تحميل QR Code"><i class="fas fa-download"></i></a>
                         @endif
                    </div>
                </div>
            </div>
            @empty
            <div class="col-12">
                <div class="alert alert-info text-center">
                    <i class="fas fa-info-circle me-2"></i>لا توجد حيوانات مسجلة بعد. <a href="{{ route('pets.create') }}">أضف حيوانك الأول!</a>
                </div>
            </div>
            @endforelse
        </div>
    </div>
    @endsection
    ```

4.  **`resources/views/pets/create.blade.php`**: (إضافة حيوان)
    ```blade
    @extends('layouts.app')

    @section('content')
    <div class="container mt-4">
        <div class="card shadow-sm">
            <div class="card-header bg-primary text-white">
                <h4 class="mb-0">إضافة حيوان أليف جديد</h4>
            </div>
            <div class="card-body">
                <form action="{{ route('pets.store') }}" method="POST" enctype="multipart/form-data">
                    @csrf
                    <div class="row g-3">
                        <div class="col-md-6">
                            <label for="name" class="form-label">الاسم <span class="text-danger">*</span></label>
                            <input type="text" class="form-control @error('name') is-invalid @enderror" id="name" name="name" value="{{ old('name') }}" required>
                            @error('name')<div class="invalid-feedback">{{ $message }}</div>@enderror
                        </div>
                        <div class="col-md-6">
                            <label for="species" class="form-label">النوع <span class="text-danger">*</span></label>
                            <select class="form-select @error('species') is-invalid @enderror" id="species" name="species" required>
                                <option value="">اختر النوع</option>
                                <option value="dog" {{ old('species') == 'dog' ? 'selected' : '' }}>كلب</option>
                                <option value="cat" {{ old('species') == 'cat' ? 'selected' : '' }}>قطة</option>
                                <option value="bird" {{ old('species') == 'bird' ? 'selected' : '' }}>طائر</option>
                                <option value="fish" {{ old('species') == 'fish' ? 'selected' : '' }}>سمكة</option>
                                <option value="livestock" {{ old('species') == 'livestock' ? 'selected' : '' }}>مواشي</option>
                                <option value="other" {{ old('species') == 'other' ? 'selected' : '' }}>أخرى</option>
                            </select>
                            @error('species')<div class="invalid-feedback">{{ $message }}</div>@enderror
                        </div>
                        <div class="col-md-6">
                            <label for="breed" class="form-label">السلالة</label>
                            <input type="text" class="form-control" id="breed" name="breed" value="{{ old('breed') }}">
                        </div>
                        <div class="col-md-6">
                            <label for="gender" class="form-label">الجنس</label>
                            <select class="form-select" id="gender" name="gender">
                                <option value="">اختر الجنس</option>
                                <option value="male" {{ old('gender') == 'male' ? 'selected' : '' }}>ذكر</option>
                                <option value="female" {{ old('gender') == 'female' ? 'selected' : '' }}>أنثى</option>
                                <option value="unknown" {{ old('gender') == 'unknown' ? 'selected' : '' }}>غير معروف</option>
                            </select>
                        </div>
                        <div class="col-md-6">
                            <label for="date_of_birth" class="form-label">تاريخ الميلاد</label>
                            <input type="date" class="form-control" id="date_of_birth" name="date_of_birth" value="{{ old('date_of_birth') }}">
                        </div>
                        <div class="col-md-6">
                            <label for="color" class="form-label">اللون</label>
                            <input type="text" class="form-control" id="color" name="color" value="{{ old('color') }}">
                        </div>
                        <div class="col-12">
                            <label for="distinguishing_marks" class="form-label">العلامات المميزة</label>
                            <textarea class="form-control" id="distinguishing_marks" name="distinguishing_marks" rows="3">{{ old('distinguishing_marks') }}</textarea>
                        </div>
                        <div class="col-md-6">
                            <label for="microchip_number" class="form-label">رقم المايكروشيب (إن وجد)</label>
                            <input type="text" class="form-control" id="microchip_number" name="microchip_number" value="{{ old('microchip_number') }}">
                        </div>
                        <div class="col-md-6">
                            <label for="image" class="form-label">صورة الحيوان</label>
                            <input class="form-control" type="file" id="image" name="image">
                        </div>
                    </div>
                    <div class="d-grid gap-2 col-6 mx-auto mt-4">
                        <button type="submit" class="btn btn-primary btn-lg"><i class="fas fa-save me-2"></i>حفظ الحيوان</button>
                    </div>
                </form>
            </div>
        </div>
    </div>
    @endsection
    ```

5.  **`resources/views/pets/edit.blade.php`**: (تعديل حيوان - مشابه لـ create)
    ```blade
    @extends('layouts.app')

    @section('content')
    <div class="container mt-4">
        <div class="card shadow-sm">
            <div class="card-header bg-primary text-white">
                <h4 class="mb-0">تعديل بيانات الحيوان الأليف: {{ $pet->name }}</h4>
            </div>
            <div class="card-body">
                <form action="{{ route('pets.update', $pet->id) }}" method="POST" enctype="multipart/form-data">
                    @csrf
                    @method('PUT')
                    <div class="row g-3">
                        <div class="col-md-6">
                            <label for="name" class="form-label">الاسم <span class="text-danger">*</span></label>
                            <input type="text" class="form-control @error('name') is-invalid @enderror" id="name" name="name" value="{{ old('name', $pet->name) }}" required>
                            @error('name')<div class="invalid-feedback">{{ $message }}</div>@enderror
                        </div>
                        <div class="col-md-6">
                            <label for="species" class="form-label">النوع <span class="text-danger">*</span></label>
                            <select class="form-select @error('species') is-invalid @enderror" id="species" name="species" required>
                                <option value="">اختر النوع</option>
                                <option value="dog" {{ old('species', $pet->species) == 'dog' ? 'selected' : '' }}>كلب</option>
                                <option value="cat" {{ old('species', $pet->species) == 'cat' ? 'selected' : '' }}>قطة</option>
                                <option value="bird" {{ old('species', $pet->species) == 'bird' ? 'selected' : '' }}>طائر</option>
                                <option value="fish" {{ old('species', $pet->species) == 'fish' ? 'selected' : '' }}>سمكة</option>
                                <option value="livestock" {{ old('species', $pet->species) == 'livestock' ? 'selected' : '' }}>مواشي</option>
                                <option value="other" {{ old('species') == 'other' ? 'selected' : '' }}>أخرى</option>
                            </select>
                            @error('species')<div class="invalid-feedback">{{ $message }}</div>@enderror
                        </div>
                        <div class="col-md-6">
                            <label for="breed" class="form-label">السلالة</label>
                            <input type="text" class="form-control" id="breed" name="breed" value="{{ old('breed', $pet->breed) }}">
                        </div>
                        <div class="col-md-6">
                            <label for="gender" class="form-label">الجنس</label>
                            <select class="form-select" id="gender" name="gender">
                                <option value="">اختر الجنس</option>
                                <option value="male" {{ old('gender', $pet->gender) == 'male' ? 'selected' : '' }}>ذكر</option>
                                <option value="female" {{ old('gender', $pet->gender) == 'female' ? 'selected' : '' }}>أنثى</option>
                                <option value="unknown" {{ old('gender', $pet->gender) == 'unknown' ? 'selected' : '' }}>غير معروف</option>
                            </select>
                        </div>
                        <div class="col-md-6">
                            <label for="date_of_birth" class="form-label">تاريخ الميلاد</label>
                            <input type="date" class="form-control" id="date_of_birth" name="date_of_birth" value="{{ old('date_of_birth', $pet->date_of_birth ? $pet->date_of_birth->format('Y-m-d') : '') }}">
                        </div>
                        <div class="col-md-6">
                            <label for="color" class="form-label">اللون</label>
                            <input type="text" class="form-control" id="color" name="color" value="{{ old('color', $pet->color) }}">
                        </div>
                        <div class="col-12">
                            <label for="distinguishing_marks" class="form-label">العلامات المميزة</label>
                            <textarea class="form-control" id="distinguishing_marks" name="distinguishing_marks" rows="3">{{ old('distinguishing_marks', $pet->distinguishing_marks) }}</textarea>
                        </div>
                        <div class="col-md-6">
                            <label for="microchip_number" class="form-label">رقم المايكروشيب (إن وجد)</label>
                            <input type="text" class="form-control" id="microchip_number" name="microchip_number" value="{{ old('microchip_number', $pet->microchip_number) }}">
                        </div>
                        <div class="col-md-6">
                            <label for="image" class="form-label">صورة الحيوان الحالية</label>
                            @if($pet->image_path)
                                <img src="{{ asset('storage/' . $pet->image_path) }}" alt="صورة {{ $pet->name }}" class="img-thumbnail mb-2" style="max-width: 150px;">
                            @endif
                            <input class="form-control" type="file" id="image" name="image">
                            <small class="text-muted">اتركها فارغة للإبقاء على الصورة الحالية.</small>
                        </div>
                    </div>
                    <div class="d-grid gap-2 col-6 mx-auto mt-4">
                        <button type="submit" class="btn btn-primary btn-lg"><i class="fas fa-sync-alt me-2"></i>تحديث البيانات</button>
                    </div>
                </form>
            </div>
        </div>
    </div>
    @endsection
    ```

6.  **`resources/views/pets/show.blade.php`**: (ملف الحيوان المفصل - يجمع بين التصاميم السابقة بشكل ديناميكي)
    ```blade
    @extends('layouts.app')

    @section('title', 'ملف ' . $pet->name)

    @section('content')
    <div class="container mt-4">
        {{-- عرض تنبيه اذا كان المستخدم ليس المالك ويدخل عبر لوحة التحكم --}}
        @if(auth()->check() && auth()->user()->id !== $pet->owner_id && !auth()->user()->isAdmin() && !auth()->user()->isDataEntry())
        <div class="alert alert-info alert-dismissible fade show" role="alert">
            <i class="fas fa-info-circle me-2"></i>أنت تشاهد هذا الملف كزائر. لن تتمكن من تعديل البيانات.
            <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
        </div>
        @endif

        <div class="row g-4">
            <!-- العمود الأيمن: الصورة والمعلومات الأساسية -->
            <div class="col-lg-4">
                <div class="card shadow-sm mb-4 sticky-lg-top">
                    <div class="card-header bg-primary text-white">
                        <h4 class="mb-0 text-center">{{ $pet->name }}
                            @if($pet->species == 'dog') <i class="fas fa-dog ms-2"></i>
                            @elseif($pet->species == 'cat') <i class="fas fa-cat ms-2"></i>
                            @elseif($pet->species == 'bird') <i class="fas fa-dove ms-2"></i>
                            @else <i class="fas fa-paw ms-2"></i>
                            @endif
                        </h4>
                    </div>
                    <img src="{{ $pet->image_path ? asset('storage/' . $pet->image_path) : 'https://via.placeholder.com/400?text=Pet+Image' }}" class="pet-profile-img p-3" alt="صورة {{ $pet->name }}">
                    <div class="card-body profile-details">
                        <dl class="row mb-0">
                            <dt class="col-sm-5"><i class="fas fa-paw me-2 text-primary"></i>النوع:</dt>
                            <dd class="col-sm-7">{{ $pet->species }}</dd>

                            <dt class="col-sm-5"><i class="fas fa-tag me-2 text-primary"></i>السلالة:</dt>
                            <dd class="col-sm-7">{{ $pet->breed ?? 'غير محدد' }}</dd>

                            <dt class="col-sm-5"><i class="fas fa-venus-mars me-2 text-primary"></i>الجنس:</dt>
                            <dd class="col-sm-7">{{ $pet->gender ?? 'غير معروف' }}</dd>

                             <dt class="col-sm-5"><i class="fas fa-palette me-2 text-primary"></i>اللون:</dt>
                            <dd class="col-sm-7">{{ $pet->color ?? 'غير محدد' }}</dd>

                            <dt class="col-sm-5"><i class="fas fa-birthday-cake me-2 text-primary"></i>تاريخ الميلاد:</dt>
                            <dd class="col-sm-7">{{ $pet->date_of_birth ? $pet->date_of_birth->format('d/m/Y') : 'غير محدد' }}</dd>

                            <dt class="col-sm-5"><i class="fas fa-barcode me-2 text-primary"></i>الرقم التسلسلي:</dt>
                            <dd class="col-sm-7"><code>{{ $pet->uuid }}</code></dd>

                             <dt class="col-sm-5"><i class="fas fa-microchip me-2 text-primary"></i>المايكروشيب:</dt>
                            <dd class="col-sm-7">{{ $pet->microchip_number ?? 'لا يوجد' }}</dd>

                            <dt class="col-sm-5"><i class="fas fa-user me-2 text-primary"></i>المالك الحالي:</dt>
                            <dd class="col-sm-7">{{ $pet->owner->name ?? 'غير محدد' }}</dd>

                            @if($pet->distinguishing_marks)
                            <dt class="col-sm-5"><i class="fas fa-pencil-alt me-2 text-primary"></i>علامات مميزة:</dt>
                            <dd class="col-sm-7">{{ $pet->distinguishing_marks }}</dd>
                            @endif
                        </dl>
                    </div>
                     <div class="card-footer text-center">
                         @if(auth()->user()->isAdmin() || auth()->user()->isDataEntry() || auth()->user()->id === $pet->owner_id)
                         <a href="{{ route('pets.edit', $pet->id) }}" class="btn btn-secondary btn-sm"><i class="fas fa-edit me-1"></i> تعديل البيانات</a>
                         @endif
                         @if($pet->qrCodeLink)
                         <a href="{{ asset('storage/' . $pet->qrCodeLink->qr_image_path) }}" target="_blank" class="btn btn-info btn-sm text-white"><i class="fas fa-qrcode me-1"></i> عرض الباركود</a>
                         @endif
                         @if(auth()->user()->isAdmin() || auth()->user()->isDataEntry())
                         <form action="{{ route('pets.destroy', $pet->id) }}" method="POST" class="d-inline">
                             @csrf
                             @method('DELETE')
                             <button type="submit" class="btn btn-danger btn-sm" onclick="return confirm('هل أنت متأكد من حذف هذا الحيوان؟ لا يمكن التراجع عن هذا الإجراء.');"><i class="fas fa-trash me-1"></i> حذف</button>
                         </form>
                         @endif
                         {{-- زر نقل الملكية --}}
                         @if(auth()->user()->isAdmin() || auth()->user()->isDataEntry() || auth()->user()->id === $pet->owner_id)
                         <button class="btn btn-warning btn-sm"><i class="fas fa-exchange-alt me-1"></i> نقل الملكية</button>
                         @endif
                     </div>
                </div>
            </div>

            <!-- العمود الأيسر: التابات (الصحي، الغذائي، إلخ) -->
            <div class="col-lg-8">
                 <div class="card shadow-sm">
                     <div class="card-header">
                         <ul class="nav nav-pills card-header-pills" id="petTab" role="tablist">
                             <li class="nav-item" role="presentation">
                                 <button class="nav-link active" id="medical-tab" data-bs-toggle="tab" data-bs-target="#medical" type="button" role="tab" aria-controls="medical" aria-selected="true"><i class="fas fa-notes-medical me-1"></i>الملف الصحي</button>
                             </li>
                             <li class="nav-item" role="presentation">
                                 <button class="nav-link" id="feeding-tab" data-bs-toggle="tab" data-bs-target="#feeding" type="button" role="tab" aria-controls="feeding" aria-selected="false"><i class="fas fa-utensils me-1"></i>الملف الغذائي</button>
                             </li>
                             <li class="nav-item" role="presentation">
                                 <button class="nav-link" id="media-tab" data-bs-toggle="tab" data-bs-target="#media" type="button" role="tab" aria-controls="media" aria-selected="false"><i class="fas fa-photo-video me-1"></i>الوسائط</button>
                             </li>
                             <li class="nav-item" role="presentation">
                                 <button class="nav-link" id="breeding-tab" data-bs-toggle="tab" data-bs-target="#breeding" type="button" role="tab" aria-controls="breeding" aria-selected="false"><i class="fas fa-baby-carriage me-1"></i>الإنتاج والتكاثر</button>
                             </li>
                         </ul>
                     </div>
                     <div class="card-body">
                         <div class="tab-content" id="petTabContent">
                             {{-- محتوى الملف الصحي --}}
                             <div class="tab-pane fade show active" id="medical" role="tabpanel" aria-labelledby="medical-tab">
                                 <h5 class="mb-3">السجل الطبي لـ "{{ $pet->name }}"</h5>
                                 @if(auth()->user()->isAdmin() || auth()->user()->isDataEntry() || auth()->user()->id === $pet->owner_id)
                                 <button class="btn btn-success btn-sm mb-3"><i class="fas fa-plus me-1"></i> إضافة سجل طبي جديد</button>
                                 @endif
                                 <h6><i class="fas fa-syringe me-2 text-info"></i>اللقاحات</h6>
                                 @if($pet->medicalRecords->where('record_type', 'vaccination')->isNotEmpty())
                                 <table class="table table-striped table-hover table-sm">
                                     <thead>
                                         <tr>
                                             <th>التاريخ</th>
                                             <th>نوع اللقاح</th>
                                             <th>الملاحظات</th>
                                             <th>المستند</th>
                                             <th>إجراء</th>
                                         </tr>
                                     </thead>
                                     <tbody>
                                         @foreach($pet->medicalRecords->where('record_type', 'vaccination')->sortByDesc('record_date') as $record)
                                         <tr>
                                             <td>{{ $record->record_date->format('d/m/Y') }}</td>
                                             <td>{{ $record->vaccine_type }}</td>
                                             <td>{{ $record->description }}</td>
                                             <td>@if($record->attachment_path)<a href="{{ asset('storage/' . $record->attachment_path) }}" target="_blank"><i class="fas fa-file-alt"></i> عرض</a>@else - @endif</td>
                                             <td>
                                                 @if(auth()->user()->isAdmin() || auth()->user()->isDataEntry() || auth()->user()->id === $pet->owner_id)
                                                 <button class="btn btn-outline-danger btn-sm py-0 px-1"><i class="fas fa-trash-alt"></i></button>
                                                 @endif
                                             </td>
                                         </tr>
                                         @endforeach
                                     </tbody>
                                 </table>
                                 @else
                                 <p class="text-muted">لا توجد لقاحات مسجلة بعد.</p>
                                 @endif
                                 <hr>
                                 <h6><i class="fas fa-allergies me-2 text-warning"></i>الحساسية</h6>
                                 @if($pet->medicalRecords->where('record_type', 'allergy')->isNotEmpty())
                                 <p><strong>غذائية:</strong> {{ $pet->medicalRecords->where('record_type', 'allergy')->where('allergy_type', 'food')->pluck('description')->implode(', ') ?? 'لا يوجد' }}</p>
                                 <p><strong>دوائية:</strong> {{ $pet->medicalRecords->where('record_type', 'allergy')->where('allergy_type', 'drug')->pluck('description')->implode(', ') ?? 'لا يوجد' }}</p>
                                 @else
                                 <p class="text-muted">لا توجد حساسيات مسجلة.</p>
                                 @endif
                                 <hr>
                                  <h6><i class="fas fa-notes-medical me-2 text-danger"></i>الأمراض المزمنة</h6>
                                 @if($pet->medicalRecords->where('record_type', 'disease')->isNotEmpty())
                                  <p>{{ $pet->medicalRecords->where('record_type', 'disease')->pluck('description')->implode(', ') }}</p>
                                 @else
                                  <p class="text-muted">لا يوجد.</p>
                                 @endif
                             </div>

                             {{-- محتوى الملف الغذائي --}}
                             <div class="tab-pane fade" id="feeding" role="tabpanel" aria-labelledby="feeding-tab">
                                 <h5 class="mb-3">النظام الغذائي لـ "{{ $pet->name }}"</h5>
                                 @if(auth()->user()->isAdmin() || auth()->user()->isDataEntry() || auth()->user()->id === $pet->owner_id)
                                  <button class="btn btn-secondary btn-sm mb-3"><i class="fas fa-edit me-1"></i> تعديل النظام الغذائي</button>
                                 @endif
                                 @if($pet->feedingRecords)
                                 <dl class="row">
                                     <dt class="col-sm-4">الغذاء الرئيسي:</dt>
                                     <dd class="col-sm-8">{{ $pet->feedingRecords->main_food ?? '-' }}</dd>
                                     <dt class="col-sm-4">عدد الوجبات:</dt>
                                     <dd class="col-sm-8">{{ $pet->feedingRecords->meals_per_day ?? '-' }}</dd>
                                     <dt class="col-sm-4">كمية الوجبة الواحدة:</dt>
                                     <dd class="col-sm-8">{{ $pet->feedingRecords->food_quantity ?? '-' }}</dd>
                                     <dt class="col-sm-4">الإضافات الغذائية:</dt>
                                     <dd class="col-sm-8">{{ $pet->feedingRecords->dietary_additions ?? '-' }}</dd>
                                     <dt class="col-sm-4">المتممات الغذائية:</dt>
                                     <dd class="col-sm-8">{{ $pet->feedingRecords->supplements ?? '-' }}</dd>
                                     <dt class="col-sm-4">الغذاء المفضل:</dt>
                                     <dd class="col-sm-8">{{ $pet->feedingRecords->favorite_food ?? '-' }}</dd>
                                     <dt class="col-sm-4">الحساسية الغذائية:</dt>
                                     <dd class="col-sm-8">{{ $pet->feedingRecords->food_allergies ?? 'لا يوجد' }}</dd>
                                 </dl>
                                 @else
                                 <p class="text-muted">لا توجد سجلات غذائية لهذا الحيوان.</p>
                                 @endif
                             </div>

                             {{-- محتوى الوسائط --}}
                              <div class="tab-pane fade" id="media" role="tabpanel" aria-labelledby="media-tab">
                                  <h5 class="mb-3">صور وفيديوهات "{{ $pet->name }}"</h5>
                                  @if(auth()->user()->isAdmin() || auth()->user()->isDataEntry() || auth()->user()->id === $pet->owner_id)
                                  <button class="btn btn-success btn-sm mb-3"><i class="fas fa-upload me-1"></i> رفع صورة/فيديو</button>
                                  @endif
                                  <div class="row g-2">
                                      @forelse($pet->media as $media)
                                      <div class="col-6 col-md-4 col-lg-3">
                                          @if($media->file_type == 'image')
                                              <img src="{{ asset('storage/' . $media->file_path) }}" class="img-thumbnail" alt="{{ $media->description }}">
                                          @else
                                              <div class="img-thumbnail d-flex justify-content-center align-items-center bg-secondary text-white" style="height: 150px;">
                                                  <i class="fas fa-play-circle fa-3x"></i>
                                                  {{-- يمكن إضافة رابط للفيديو هنا --}}
                                              </div>
                                          @endif
                                          <p class="small text-muted text-center mt-1">{{ $media->description }}</p>
                                      </div>
                                      @empty
                                      <div class="col-12"><p class="text-muted">لا توجد وسائط مضافة بعد.</p></div>
                                      @endforelse
                                  </div>
                              </div>

                             {{-- محتوى الإنتاج --}}
                             <div class="tab-pane fade" id="breeding" role="tabpanel" aria-labelledby="breeding-tab">
                                 <h5 class="mb-3">سجل الإنتاج والتكاثر</h5>
                                 @if(auth()->user()->isAdmin() || auth()->user()->isDataEntry() || auth()->user()->id === $pet->owner_id)
                                  <button class="btn btn-info btn-sm mb-3 text-white"><i class="fas fa-plus me-1"></i> إضافة سجل تزاوج/ولادة</button>
                                 @endif
                                 @if($pet->breedingRecords->isNotEmpty())
                                 <table class="table table-sm table-bordered">
                                     <thead>
                                         <tr>
                                             <th>الشريك</th>
                                             <th>تاريخ التزاوج/الحمل</th>
                                             <th>تاريخ الولادة/الفقس</th>
                                             <th>عدد المواليد/البيض</th>
                                             <th>ملاحظات</th>
                                         </tr>
                                     </thead>
                                     <tbody>
                                         @foreach($pet->breedingRecords as $record)
                                         <tr>
                                             <td>{{ $record->mate ? $record->mate->name : 'غير محدد' }}</td>
                                             <td>{{ $record->pairing_date ?? $record->pregnancy_date ?? '-' }}</td>
                                             <td>{{ $record->birth_date ?? $record->hatch_date ?? '-' }}</td>
                                             <td>{{ $record->num_offspring ?? $record->num_eggs ?? '-' }}</td>
                                             <td>{{ $record->notes ?? '-' }}</td>
                                         </tr>
                                         @endforeach
                                     </tbody>
                                 </table>
                                 @else
                                 <p class="text-muted">لا توجد سجلات إنتاج حالياً لهذا الحيوان.</p>
                                 @endif
                             </div>
                         </div>
                     </div>
                 </div>
            </div>
        </div>
    </div>
    @endsection
    ```

7.  **`resources/views/pets/public_profile.blade.php`**: (الملف العام للحيوان - للقراءة فقط)
    ```blade
    @extends('layouts.app')

    @section('title', 'ملف ' . $pet->name . ' (عام)')

    @section('content')
    <div class="container mt-4">
        <div class="alert alert-info alert-dismissible fade show" role="alert">
            <i class="fas fa-qrcode me-2"></i>أنت تعرض هذا الملف كزائر عام (عبر الباركود). بعض المعلومات قد تكون محدودة.
            <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
        </div>

        <div class="row g-4">
            <div class="col-lg-4">
                <div class="card shadow-sm mb-4 sticky-lg-top">
                    <div class="card-header bg-primary text-white">
                        <h4 class="mb-0 text-center">{{ $pet->name }}
                            @if($pet->species == 'dog') <i class="fas fa-dog ms-2"></i>
                            @elseif($pet->species == 'cat') <i class="fas fa-cat ms-2"></i>
                            @elseif($pet->species == 'bird') <i class="fas fa-dove ms-2"></i>
                            @else <i class="fas fa-paw ms-2"></i>
                            @endif
                        </h4>
                    </div>
                    <img src="{{ $pet->image_path ? asset('storage/' . $pet->image_path) : 'https://via.placeholder.com/400?text=Pet+Image' }}" class="pet-profile-img p-3" alt="صورة {{ $pet->name }}">
                    <div class="card-body profile-details">
                        <dl class="row mb-0">
                            <dt class="col-sm-5"><i class="fas fa-paw me-2 text-primary"></i>النوع:</dt>
                            <dd class="col-sm-7">{{ $pet->species }}</dd>

                            <dt class="col-sm-5"><i class="fas fa-tag me-2 text-primary"></i>السلالة:</dt>
                            <dd class="col-sm-7">{{ $pet->breed ?? 'غير محدد' }}</dd>

                            <dt class="col-sm-5"><i class="fas fa-venus-mars me-2 text-primary"></i>الجنس:</dt>
                            <dd class="col-sm-7">{{ $pet->gender ?? 'غير معروف' }}</dd>

                             <dt class="col-sm-5"><i class="fas fa-palette me-2 text-primary"></i>اللون:</dt>
                            <dd class="col-sm-7">{{ $pet->color ?? 'غير محدد' }}</dd>

                            <dt class="col-sm-5"><i class="fas fa-birthday-cake me-2 text-primary"></i>تاريخ الميلاد:</dt>
                            <dd class="col-sm-7">{{ $pet->date_of_birth ? $pet->date_of_birth->format('d/m/Y') : 'غير محدد' }}</dd>

                            <dt class="col-sm-5"><i class="fas fa-barcode me-2 text-primary"></i>الرقم التسلسلي:</dt>
                            <dd class="col-sm-7"><code>{{ $pet->uuid }}</code></dd>

                             <dt class="col-sm-5"><i class="fas fa-microchip me-2 text-primary"></i>المايكروشيب:</dt>
                            <dd class="col-sm-7">{{ $pet->microchip_number ?? 'لا يوجد' }}</dd>
                            {{-- يمكن إخفاء معلومات المالك لأسباب الخصوصية هنا --}}
                            {{-- <dt class="col-sm-5"><i class="fas fa-user me-2 text-primary"></i>المالك الحالي:</dt>
                            <dd class="col-sm-7">{{ $pet->owner->name ?? 'غير محدد' }}</dd> --}}
                        </dl>
                    </div>
                </div>
            </div>

            <div class="col-lg-8">
                 <div class="card shadow-sm">
                     <div class="card-header">
                         <ul class="nav nav-pills card-header-pills" id="petTab" role="tablist">
                             <li class="nav-item" role="presentation">
                                 <button class="nav-link active" id="medical-tab" data-bs-toggle="tab" data-bs-target="#medical" type="button" role="tab" aria-controls="medical" aria-selected="true"><i class="fas fa-notes-medical me-1"></i>الملف الصحي</button>
                             </li>
                             <li class="nav-item" role="presentation">
                                 <button class="nav-link" id="feeding-tab" data-bs-toggle="tab" data-bs-target="#feeding" type="button" role="tab" aria-controls="feeding" aria-selected="false"><i class="fas fa-utensils me-1"></i>الملف الغذائي</button>
                             </li>
                             <li class="nav-item" role="presentation">
                                 <button class="nav-link" id="media-tab" data-bs-toggle="tab" data-bs-target="#media" type="button" role="tab" aria-controls="media" aria-selected="false"><i class="fas fa-photo-video me-1"></i>الوسائط</button>
                             </li>
                             {{-- لا نعرض سجل الإنتاج للعامة غالباً --}}
                         </ul>
                     </div>
                     <div class="card-body">
                         <div class="tab-content" id="petTabContent">
                             {{-- محتوى الملف الصحي (نسخة للقراءة فقط) --}}
                             <div class="tab-pane fade show active" id="medical" role="tabpanel" aria-labelledby="medical-tab">
                                 <h5 class="mb-3">السجل الطبي لـ "{{ $pet->name }}"</h5>
                                 <h6><i class="fas fa-syringe me-2 text-info"></i>اللقاحات</h6>
                                 @if($pet->medicalRecords->where('record_type', 'vaccination')->isNotEmpty())
                                 <table class="table table-striped table-hover table-sm">
                                     <thead>
                                         <tr>
                                             <th>التاريخ</th>
                                             <th>نوع اللقاح</th>
                                             <th>الملاحظات</th>
                                         </tr>
                                     </thead>
                                     <tbody>
                                         @foreach($pet->medicalRecords->where('record_type', 'vaccination')->sortByDesc('record_date') as $record)
                                         <tr>
                                             <td>{{ $record->record_date->format('d/m/Y') }}</td>
                                             <td>{{ $record->vaccine_type }}</td>
                                             <td>{{ $record->description }}</td>
                                         </tr>
                                         @endforeach
                                     </tbody>
                                 </table>
                                 @else
                                 <p class="text-muted">لا توجد لقاحات مسجلة بعد.</p>
                                 @endif
                                 <hr>
                                 <h6><i class="fas fa-allergies me-2 text-warning"></i>الحساسية</h6>
                                 @if($pet->medicalRecords->where('record_type', 'allergy')->isNotEmpty())
                                 <p><strong>غذائية:</strong> {{ $pet->medicalRecords->where('record_type', 'allergy')->where('allergy_type', 'food')->pluck('description')->implode(', ') ?? 'لا يوجد' }}</p>
                                 <p><strong>دوائية:</strong> {{ $pet->medicalRecords->where('record_type', 'allergy')->where('allergy_type', 'drug')->pluck('description')->implode(', ') ?? 'لا يوجد' }}</p>
                                 @else
                                 <p class="text-muted">لا توجد حساسيات مسجلة.</p>
                                 @endif
                                 <hr>
                                  <h6><i class="fas fa-notes-medical me-2 text-danger"></i>الأمراض المزمنة</h6>
                                 @if($pet->medicalRecords->where('record_type', 'disease')->isNotEmpty())
                                  <p>{{ $pet->medicalRecords->where('record_type', 'disease')->pluck('description')->implode(', ') }}</p>
                                 @else
                                  <p class="text-muted">لا يوجد.</p>
                                 @endif
                             </div>

                             {{-- محتوى الملف الغذائي (نسخة للقراءة فقط) --}}
                             <div class="tab-pane fade" id="feeding" role="tabpanel" aria-labelledby="feeding-tab">
                                 <h5 class="mb-3">النظام الغذائي لـ "{{ $pet->name }}"</h5>
                                 @if($pet->feedingRecords)
                                 <dl class="row">
                                     <dt class="col-sm-4">الغذاء الرئيسي:</dt>
                                     <dd class="col-sm-8">{{ $pet->feedingRecords->main_food ?? '-' }}</dd>
                                     <dt class="col-sm-4">عدد الوجبات:</dt>
                                     <dd class="col-sm-8">{{ $pet->feedingRecords->meals_per_day ?? '-' }}</dd>
                                     <dt class="col-sm-4">كمية الوجبة الواحدة:</dt>
                                     <dd class="col-sm-8">{{ $pet->feedingRecords->food_quantity ?? '-' }}</dd>
                                     <dt class="col-sm-4">الإضافات الغذائية:</dt>
                                     <dd class="col-sm-8">{{ $pet->feedingRecords->dietary_additions ?? '-' }}</dd>
                                     <dt class="col-sm-4">المتممات الغذائية:</dt>
                                     <dd class="col-sm-8">{{ $pet->feedingRecords->supplements ?? '-' }}</dd>
                                     <dt class="col-sm-4">الغذاء المفضل:</dt>
                                     <dd class="col-sm-8">{{ $pet->feedingRecords->favorite_food ?? '-' }}</dd>
                                     <dt class="col-sm-4">الحساسية الغذائية:</dt>
                                     <dd class="col-sm-8">{{ $pet->feedingRecords->food_allergies ?? 'لا يوجد' }}</dd>
                                 </dl>
                                 @else
                                 <p class="text-muted">لا توجد سجلات غذائية لهذا الحيوان.</p>
                                 @endif
                             </div>

                             {{-- محتوى الوسائط (نسخة للقراءة فقط) --}}
                              <div class="tab-pane fade" id="media" role="tabpanel" aria-labelledby="media-tab">
                                  <h5 class="mb-3">صور وفيديوهات "{{ $pet->name }}"</h5>
                                  <div class="row g-2">
                                      @forelse($pet->media as $media)
                                      <div class="col-6 col-md-4 col-lg-3">
                                          @if($media->file_type == 'image')
                                              <img src="{{ asset('storage/' . $media->file_path) }}" class="img-thumbnail" alt="{{ $media->description }}">
                                          @else
                                              <div class="img-thumbnail d-flex justify-content-center align-items-center bg-secondary text-white" style="height: 150px;">
                                                  <i class="fas fa-play-circle fa-3x"></i>
                                              </div>
                                          @endif
                                          <p class="small text-muted text-center mt-1">{{ $media->description }}</p>
                                      </div>
                                      @empty
                                      <div class="col-12"><p class="text-muted">لا توجد وسائط مضافة بعد.</p></div>
                                      @endforelse
                                  </div>
                              </div>
                         </div>
                     </div>
                 </div>
            </div>
        </div>
    </div>
    @endsection
    ```

8.  **`resources/views/tasks.blade.php`**: (المهام اليومية)
    ```blade
    @extends('layouts.app')

    @section('content')
    <div class="container mt-4">
        <div class="d-flex justify-content-between align-items-center mb-4">
            <h1 class="display-6 fw-light">المهام اليومية والعناية</h1>
            <button class="btn btn-primary"><i class="fas fa-plus me-1"></i> إضافة مهمة جديدة</button>
        </div>
        <hr>

        <div class="mb-3">
             <label for="taskFilter" class="form-label small">عرض مهام لـ:</label>
             <select class="form-select form-select-sm w-auto d-inline-block" id="taskFilter">
                <option selected>جميع الحيوانات</option>
                <option value="1">بادي (Buddy)</option>
                <option value="2">لوسي (Lucy)</option>
                <option value="3">كوكو (Coco)</option>
             </select>
             <label for="taskDateFilter" class="form-label ms-2">التاريخ:</label>
             <input type="date" class="form-control form-control-sm w-auto d-inline-block" id="taskDateFilter" value="2023-11-20">
        </div>

         <div>
             <div class="task-item completed d-flex justify-content-between align-items-center">
                 <div>
                     <h5 class="mb-1"><i class="fas fa-utensils me-2"></i>إطعام الصباح - بادي</h5>
                     <small class="text-muted">الوقت: 8:00 صباحًا | الأهمية: عالية</small>
                 </div>
                 <div>
                     <button class="btn btn-sm btn-outline-secondary"><i class="fas fa-undo"></i></button>
                     <span class="badge bg-success ms-2"><i class="fas fa-check"></i> مكتمل</span>
                 </div>
             </div>

             <div class="task-item d-flex justify-content-between align-items-center">
                 <div>
                     <h5 class="mb-1"><i class="fas fa-walking me-2"></i>تمشية المساء - بادي</h5>
                     <small class="text-muted">الوقت: 6:00 مساءً | الأهمية: متوسطة</small>
                 </div>
                 <div>
                      <button class="btn btn-sm btn-success"><i class="fas fa-check"></i> إنجاز</button>
                      <button class="btn btn-sm btn-outline-warning ms-1"><i class="fas fa-edit"></i></button>
                       <button class="btn btn-sm btn-outline-danger ms-1"><i class="fas fa-trash"></i></button>
                 </div>
             </div>

              <div class="task-item d-flex justify-content-between align-items-center">
                 <div>
                     <h5 class="mb-1"><i class="fas fa-box me-2"></i>تنظيف صندوق الرمل - لوسي</h5>
                     <small class="text-muted">الوقت: أي وقت اليوم | الأهمية: متوسطة</small>
                 </div>
                 <div>
                      <button class="btn btn-sm btn-success"><i class="fas fa-check"></i> إنجاز</button>
                      <button class="btn btn-sm btn-outline-warning ms-1"><i class="fas fa-edit"></i></button>
                       <button class="btn btn-sm btn-outline-danger ms-1"><i class="fas fa-trash"></i></button>
                 </div>
             </div>

              <div class="task-item d-flex justify-content-between align-items-center" style="border-left-color: #fd7e14;">
                 <div>
                     <h5 class="mb-1"><i class="fas fa-calendar-alt me-2"></i>موعد الطبيب البيطري - كوكو</h5>
                     <small class="text-muted">التاريخ: 25/11/2023 | الوقت: 11:00 صباحًا</small>
                 </div>
                  <div>
                      <span class="badge bg-warning text-dark">موعد قادم</span>
                      <button class="btn btn-sm btn-outline-warning ms-1"><i class="fas fa-edit"></i></button>
                       <button class="btn btn-sm btn-outline-danger ms-1"><i class="fas fa-trash"></i></button>
                 </div>
             </div>

         </div>

    </div>
    @endsection
    ```

9.  **`resources/views/health_alerts.blade.php`**: (التنبيهات الصحية)
    ```blade
    @extends('layouts.app')

    @section('content')
    <div class="container mt-4">
        <div class="d-flex justify-content-between align-items-center mb-3">
            <h1 class="display-6 fw-light"><i class="fas fa-bell text-danger me-2"></i>التنبيهات الصحية الهامة</h1>
             <button class="btn btn-outline-secondary btn-sm"><i class="fas fa-history me-1"></i> عرض سجل التنبيهات</button>
        </div>
        <hr class="mb-4">

        <div class="filter-section row g-2 align-items-end">
            <div class="col-md-4">
                 <label for="filterPet" class="form-label small">تصفية حسب الحيوان:</label>
                 <select class="form-select form-select-sm" id="filterPet">
                    <option selected>جميع الحيوانات</option>
                    <option value="1">بادي (Buddy)</option>
                    <option value="2">لوسي (Lucy)</option>
                    <option value="3">كوكو (Coco)</option>
                 </select>
            </div>
             <div class="col-md-4">
                 <label for="filterType" class="form-label small">تصفية حسب النوع:</label>
                 <select class="form-select form-select-sm" id="filterType">
                    <option selected>جميع الأنواع</option>
                    <option value="vaccine">تطعيمات</option>
                    <option value="appointment">مواعيد</option>
                    <option value="medication">أدوية</option>
                    <option value="checkup">فحوصات</option>
                 </select>
            </div>
             <div class="col-md-3">
                 <label for="filterUrgency" class="form-label small">تصفية حسب الأهمية:</label>
                 <select class="form-select form-select-sm" id="filterUrgency">
                    <option selected>الكل</option>
                    <option value="high">عاجل</option>
                    <option value="medium">متوسط</option>
                    <option value="low">منخفض</option>
                 </select>
            </div>
             <div class="col-md-1">
                 <button class="btn btn-primary btn-sm w-100"><i class="fas fa-filter"></i></button>
             </div>
        </div>

        <div class="mt-4">

            <div class="card alert-card shadow-sm mb-3 border-danger">
                <div class="card-body">
                    <div class="d-flex align-items-center mb-2">
                         <i class="fas fa-dog pet-icon text-primary"></i>
                         <h5 class="card-title mb-0 flex-grow-1">
                            <i class="fas fa-syringe text-danger"></i>موعد تطعيم قادم لـ <a href="#" class="text-decoration-none">بادي</a>
                        </h5>
                        <span class="badge bg-danger">عاجل</span>
                    </div>
                    <p class="card-text mb-1">نوع التطعيم: جرعة السعار (Rabies) السنوية.</p>
                    <p class="card-text"><small class="text-muted"><i class="far fa-calendar-alt me-1"></i>التاريخ المستحق: 25 نوفمبر 2023 (خلال 5 أيام)</small></p>
                </div>
                 <div class="card-footer text-end">
                     <button class="btn btn-outline-success btn-sm"><i class="fas fa-check me-1"></i> تم التطعيم</button>
                     <button class="btn btn-outline-secondary btn-sm"><i class="fas fa-calendar-plus me-1"></i> إضافة للمهام</button>
                     <button class="btn btn-outline-primary btn-sm"><i class="fas fa-eye me-1"></i> عرض ملف بادي</button>
                 </div>
            </div>

            <div class="card alert-card shadow-sm mb-3 border-warning">
                <div class="card-body">
                    <div class="d-flex align-items-center mb-2">
                         <i class="fas fa-dove pet-icon" style="color: #2ecc71;"></i>
                         <h5 class="card-title mb-0 flex-grow-1">
                            <i class="fas fa-stethoscope text-warning"></i>موعد فحص لـ <a href="#" class="text-decoration-none">كوكو</a>
                         </h5>
                        <span class="badge bg-warning text-dark">متوسط</span>
                    </div>
                    <p class="card-text mb-1">السبب: فحص دوري للطائر الصغير.</p>
                    <p class="card-text"><small class="text-muted"><i class="far fa-calendar-alt me-1"></i>التاريخ: 28 نوفمبر 2023 | <i class="far fa-clock me-1"></i>الوقت: 11:00 صباحًا</small></p>
                </div>
                 <div class="card-footer text-end">
                    <button class="btn btn-outline-info btn-sm"><i class="fas fa-map-marker-alt me-1"></i> تفاصيل العيادة</button>
                     <button class="btn btn-outline-secondary btn-sm"><i class="fas fa-calendar-plus me-1"></i> إضافة للمهام</button>
                     <button class="btn btn-outline-danger btn-sm"><i class="fas fa-times me-1"></i> إلغاء الموعد</button>
                 </div>
            </div>
        </div>
    </div>
    @endsection
    ```

10. **`resources/views/qrcodes/generate.blade.php`**: (توليد QR Codes)
    ```blade
    @extends('layouts.app')

    @section('content')
    <div class="container mt-4">
        <div class="card shadow-sm">
            <div class="card-header bg-primary text-white">
                <h4 class="mb-0"><i class="fas fa-qrcode me-2"></i>توليد رموز QR للحيوانات</h4>
            </div>
            <div class="card-body">
                <p class="text-muted">يمكنك توليد رموز QR لحيوانات موجودة أو لعدد معين من الحيوانات الجديدة غير المسماة.</p>
                <hr>
                <form action="{{ route('qrcodes.generate.submit') }}" method="POST">
                    @csrf
                    <div class="mb-3">
                        <label for="generate_type" class="form-label">طريقة التوليد:</label>
                        <select class="form-select" id="generate_type" name="generate_type">
                            <option value="new_pets" selected>توليد لحيوانات جديدة غير مسماة</option>
                            <option value="existing_pets">توليد لحيوانات موجودة (التي لا تملك رمز QR)</option>
                        </select>
                    </div>

                    <div id="new_pets_options">
                        <div class="mb-3">
                            <label for="num_qrcodes" class="form-label">عدد رموز QR المراد توليدها لحيوانات جديدة:</label>
                            <input type="number" class="form-control" id="num_qrcodes" name="num_qrcodes" min="1" max="100" value="1">
                        </div>
                    </div>

                    <div id="existing_pets_options" style="display: none;">
                        <div class="mb-3">
                            <label for="selected_pets" class="form-label">اختر الحيوانات الموجودة (Ctrl+Click لتحديد أكثر من واحد):</label>
                            <select multiple class="form-select" id="selected_pets" name="selected_pets[]" size="5">
                                @forelse($pets as $pet)
                                    @if(!$pet->qrCodeLink)
                                    <option value="{{ $pet->id }}">{{ $pet->name }} ({{ $pet->species }})</option>
                                    @endif
                                @empty
                                    <option value="" disabled>لا توجد حيوانات بدون رمز QR حالياً.</option>
                                @endforelse
                            </select>
                            <small class="text-muted">سيتم عرض الحيوانات التي لا تملك رمز QR بالفعل.</small>
                        </div>
                    </div>

                    <div class="d-grid mt-4">
                        <button type="submit" class="btn btn-primary btn-lg"><i class="fas fa-cogs me-2"></i>ابدأ التوليد</button>
                    </div>
                </form>
            </div>
        </div>
    </div>

    @push('scripts')
    <script>
        document.addEventListener('DOMContentLoaded', function() {
            const generateTypeSelect = document.getElementById('generate_type');
            const newPetsOptions = document.getElementById('new_pets_options');
            const existingPetsOptions = document.getElementById('existing_pets_options');

            function toggleOptions() {
                if (generateTypeSelect.value === 'new_pets') {
                    newPetsOptions.style.display = 'block';
                    existingPetsOptions.style.display = 'none';
                    document.getElementById('selected_pets').removeAttribute('required');
                    document.getElementById('num_qrcodes').setAttribute('required', 'required');
                } else {
                    newPetsOptions.style.display = 'none';
                    existingPetsOptions.style.display = 'block';
                    document.getElementById('selected_pets').setAttribute('required', 'required');
                    document.getElementById('num_qrcodes').removeAttribute('required');
                }
            }

            generateTypeSelect.addEventListener('change', toggleOptions);
            toggleOptions(); // Set initial state
        });
    </script>
    @endpush
    @endsection
    ```

11. **`resources/views/qrcodes/index.blade.php`**: (عرض QR Codes المولدة)
    ```blade
    @extends('layouts.app')

    @section('content')
    <div class="container mt-4">
        <div class="card shadow-sm">
            <div class="card-header bg-primary text-white">
                <h4 class="mb-0"><i class="fas fa-qrcode me-2"></i>رموز QR المولدة</h4>
            </div>
            <div class="card-body">
                @if (empty($generatedQRs))
                    <div class="alert alert-info text-center">
                        <i class="fas fa-info-circle me-2"></i>لم يتم توليد أي رموز QR في هذه الجلسة.
                        <a href="{{ route('qrcodes.generate.form') }}">انتقل إلى صفحة التوليد.</a>
                    </div>
                @else
                    <p class="mb-4">تم توليد الرموز التالية. يمكنك تنزيلها أو استخدامها.</p>
                    <div class="row g-3">
                        @foreach ($generatedQRs as $qr)
                        <div class="col-md-4 col-lg-3 text-center">
                            <div class="card h-100">
                                <div class="card-body">
                                    <h6 class="card-title">{{ $qr['pet']->name }}</h6>
                                    <img src="{{ $qr['qr_path'] }}" alt="QR Code for {{ $qr['pet']->name }}" class="img-fluid mb-2" style="max-width: 150px;">
                                    <p class="small text-muted">{{ $qr['pet']->uuid }}</p>
                                    <a href="{{ $qr['qr_path'] }}" download="{{ $qr['pet']->name }}_qrcode.png" class="btn btn-sm btn-success"><i class="fas fa-download me-1"></i>تحميل</a>
                                    <a href="{{ route('pets.show.public', ['uuid' => $qr['pet']->uuid]) }}" target="_blank" class="btn btn-sm btn-info text-white mt-2 mt-md-0 ms-md-1"><i class="fas fa-eye me-1"></i>عرض عام</a>
                                </div>
                            </div>
                        </div>
                        @endforeach
                    </div>
                @endif
            </div>
        </div>
    </div>
    @endsection
    ```

---

**الخطوة 9: تفعيل Storage Link**

لضمان وصول الـ Public إلى الصور والـ QR Codes:

```bash
php artisan storage:link
```
هذا سينشئ رابطاً رمزياً من `public/storage` إلى `storage/app/public`.

---

**الخطوة 10: البدء بخادم Laravel**

```bash
php artisan serve
```
ثم افتح متصفحك على `http://127.0.0.1:8000`.

**اختبار النظام:**

1.  **المستخدمون:**
    *   **Admin:** `admin@example.com` / `password`
    *   **Data Entry:** `dataentry@example.com` / `password`
    *   **Regular User:** `user@example.com` / `password`
2.  **اختبار الصلاحيات:**
    *   حاول الدخول بحساب `user@example.com`. يجب أن يرى "حيواناتي" لكن لا يمكنه توليد QR Codes.
    *   حاول الدخول بحساب `admin@example.com` أو `dataentry@example.com`. يجب أن يتمكن من رؤية "إدارة الباركود" وإضافة/تعديل/حذف الحيوانات.
3.  **توليد QR Code:**
    *   سجل الدخول كـ `admin` أو `dataentry`.
    *   اذهب إلى "إدارة الباركود".
    *   قم بتوليد بعض رموز QR (لحيوانات جديدة أو اختر حيوانات موجودة).
    *   بعد التوليد، سترى الرموز في الصفحة. انسخ رابط "عرض عام" لأحدها.
    *   سجل الخروج.
    *   افتح الرابط الذي نسخته. يجب أن ترى ملف الحيوان للقراءة فقط.

هذا يمثل مشروع Laravel كاملاً يعمل، يغطي جميع متطلباتك الأساسية من المصادقة، الأدوار، إدارة البيانات، وتوليد QR Codes، باستخدام تصاميمك الأمامية. يمكنك البناء على هذا الأساس لتطوير الميزات الأخرى.
---


#   d r _ e m _ d b  
 