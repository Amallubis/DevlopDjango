# DevlopDjango
Update dan Insatall kebutuhan development
sudo apt-get update
sudo apt-get install python3-pip python3-dev nginx
Upgrade pip & install virtualenv
sudo -H pip3 install --upgrade pip
sudo -H pip3 install virtualenv
Clone projek aplikasi dari github
git clone https://github.com/writerlab/aplikasi
Buat virtual env.
cd aplikasi
virtualenv .env
aktifkan lingkungan virtualnya
source .env/bin/activate
Install packages
pip install django==2.2.12 gunicorn pillow django-import-export
Konfigurasi host
nano perpus/settings.py
isi ALLOWED_HOST dengan ip atau nama domain. contoh,
ALLOWED_HOST = ['192.168.100.19']
Konfigurasi static files
ubah staticfilesdirs menjadi STATIC_ROOT
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'static/')
kumpulkan semua konten static kelokasi dir yang sudah di konfig
python manage.py collectstatic
Buat exception untuk port 8000
sudo ufw allow 8000
tes
python manage.py runserver 0.0.0.0:8000
buka browser
http://192.168.100.19:8000
tes gunicorn untuk melayani projek kita
gunicorn --bind 0.0.0.0:8000 perpus.wsgi
sampai sini konfig django app selesai
keluar dari virtualenv
deactivate
Buat file service gunicorn
sudo nano /etc/systemd/system/gunicorn.service
copy-paste
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=zulx
Group=www-data
WorkingDirectory=/home/apl/aplikasi
ExecStart=/home/zulx/perpus/.env/bin/gunicorn --access-logfile - --workers 3 --bind unix:/home/apl/aplikasi/aplikasi.sock aplikasi.wsgi:application

[Install]
WantedBy=multi-user.target
start service gunicorn
sudo systemctl start gunicorn
sudo systemctl enable gunicorn
cek status socket gunicorn
file perpus.sock akan muncul di direktori aplikasi/aplikasi.sock
sudo systemctl status gunicorn
jika ada perubahan difile service gunicorn,
maka wajib reload dan restrat service. caranya,
sudo systemctl daemon-reaload
sudo systemctl restart gunicorn
Konfigurasi Nginx untuk diteruskan ke gunicorn
sudo nano /etc/nginx/sites-available/aplikasi
copy-paste
server {
    listen 80;
    server_name 192.168.100.19;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/apl/aplikasi;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/zulx/perpus/perpus.sock;
    }
}
enable-kan dengan cara membuat link dari file perpus ke direktori site-enabled
sudo ln -s /etc/nginx/sites-available/aplikasi /etc/nginx/sites-enabled
tes konfigurasi nginx
sudo nginx -t
restart nginx
sudo systemctl restart nginx
buka firewall untuk akses normal di port 80
karena kita tidak akan menggunakan server development,
maka kita hapus rule untuk membuat port 8000
sudo ufw delete allow 8000
sudo ufw allow 'Nginx Full'
MODE DEBUG
ubah nilainya menjadi False jika sedang dalam mode Production. dan hapus if settings.DEBUG di urls.py menjadi

...
urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
