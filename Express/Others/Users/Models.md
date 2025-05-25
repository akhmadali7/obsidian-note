``` javascript
/** @namefile  user.model */
import mongoose from 'mongoose';  
import bcrypt from 'bcryptjs';  
  
const userSchema = mongoose.Schema({  
        name: {  
            type: String,  
            required: true  
        },  
        email: {  
            type: String,  
            required: true,  
            unique: true  
        },  
        password: {  
            type: String,  
            required: true  
        },  
  
    }, {timestamps: true}  
)  
  
userSchema.pre('save', async function (next) {  
    if (!this.isModified('password')) {  
        next()  
    }  
    const salt = await bcrypt.genSalt(10);  
    this.password = await bcrypt.hash(this.password, salt)  
})  
  
const User = mongoose.model('User', userSchema)  
  
export default User
```

1.  Membuat Skema (Schema) untuk User
	Mendefinisikan struktur data **User** dalam database MongoDB menggunakan **mongoose.Schema**:
	- **`name`** → Tipe `String`, wajib diisi (`required: true`).
	- **`email`** → Tipe `String`, wajib diisi (`required: true`), dan harus unik (`unique: true`), artinya tidak boleh ada email yang sama dalam database.
	- **`password`** → Tipe `String`, wajib diisi.
	- **`timestamps: true`** → Secara otomatis menambahkan kolom `createdAt` dan `updatedAt` untuk mengetahui kapan data dibuat atau diperbarui.
	
2. Middleware: Mengenkripsi Password Sebelum Disimpan
	- **`pre('save', callback)`** → Ini adalah middleware yang dijalankan **sebelum data user disimpan** ke database.
	-  **`if (!this.isModified('password')) next();`** Jika password **tidak berubah**, maka lanjutkan (`next()`) tanpa mengenkripsi lagi. Ini berguna untuk **mencegah hashing ulang** ketika user hanya mengupdate data selain password.
    - **`const salt = await bcrypt.genSalt(10);`** Membuat **salt** (kode tambahan) untuk keamanan ekstra sebelum hashing.
    - **`this.password = await bcrypt.hash(this.password, salt);`** Mengubah password menjadi **hash** sebelum disimpan ke database agar lebih aman.
    
3. Membuat Model User
	``` javascript
	const User = mongoose.model('User', userSchema)  
	```
	- **`mongoose.model('User', userSchema)`** → Membuat model `User` berdasarkan skema `userSchema`.
	- Model ini akan digunakan di tempat lain untuk membuat, membaca, memperbarui, atau menghapus data user.