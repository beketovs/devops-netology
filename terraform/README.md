Будут игнорироваться:
1. Все файлы в директории .terraform
2. Все файлы в названии которых содержится "tfstate"
3. Все файлы с расширением .log и в названии которых содержится "crash"
4. Все файлы с разрешением .tfvars
5. Файлы override.tf и override.tf.json , а также файлы, которые заканчиваются на _override.tf и _override.tf.json
6. Файлы с разрешением .terraformrc и terraform.rc 
