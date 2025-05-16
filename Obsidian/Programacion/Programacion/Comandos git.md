credenciales de git cambiar el origin.url

```git 

git config --local --list

```


cambiar la url de la ruta de origen por si no esta usando la de otro y por eso salen las notificaciones de logeo de git

```git

git remote set-url origin https://gmCagigas88@bitbucket.org/incentro-cloud/cargonaut-be.git

```


eliminar el ultimo commit 
```git

git reset --hard HEAD~1  
git push origin HEAD --force

```

