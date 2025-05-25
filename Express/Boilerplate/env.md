``` .env
# Port on which the app will run  
PORT=4000  
  
# Database connection settings  
DB_HOST=localhost  
DB_USER=root  
DB_PASSWORD=Aciciacil77.  
DB_NAME=DSSVikor  
  
# Bcrypt salt rounds for password hashing  
BCRYPT_SALT_ROUNDS=10  
  
# JWT secrets (replace with your actual secrets in production)  
JWT_SECRET_ACCESS_TOKEN=KMNOPQ1234ABCD  
JWT_SECRET_REFRESH_TOKEN=1234ABCDKMNOPQ  
JWT_SECRET_EMAIL_VERIFICATION=ABCD1234KMNOPQ  
  
# JWT token expiration settings (adjust as needed)  
JWT_ACCESS_TOKEN_EXPIRES_IN='30m'  
JWT_REFRESH_TOKEN_EXPIRES_IN='30d'  
JWT_EMAIL_VERIFICATION_EXPIRES_IN='1h'  
  
# Email settings  
EMAIL_USER=muhammadthanos@gmail.com  
EMAIL_PASS=cpbppnjfwzlbzuyz
```