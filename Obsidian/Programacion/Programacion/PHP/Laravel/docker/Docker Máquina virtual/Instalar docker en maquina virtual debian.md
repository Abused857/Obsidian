

# Eliminar repositorio incorrecto de ubunto en maquina virtual debian

>sudo rm /etc/apt/sources.list.d/docker.list


# set up docker repository en la maquina virtual

# Actualiza la lista de paquetes de tu sistema:

>sudo apt-get update

# Instala las dependencias necesarias para Docker:

>sudo apt-get install ca-certificates curl gnupg

# Crea un directorio para almacenar la clave GPG y luego agrégala:

>sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo tee /etc/apt/keyrings/docker.asc > /dev/null
sudo chmod a+r /etc/apt/keyrings/docker.asc


# Ahora añade el repositorio de Docker para Debian Bookworm:

>echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Ejecuta el siguiente comando para actualizar la lista de paquetes de nuevo:

>sudo apt-get update

# Ahora puedes instalar Docker y sus componentes:

>sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Comprueba si Docker se instaló correctamente ejecutando el siguiente comando:

>sudo docker --version

# Para asegurarte de que Docker se ejecute automáticamente después de cada reinicio, habilítalo e inícialo:

>sudo systemctl enable docker
sudo systemctl start docker

# instalar el complemento de Docker Compose:

>sudo apt-get update
sudo apt-get install docker-compose-plugin

# verificar que todo funcione correctamente:

>docker compose version



[[docker Laravel]] [[Install git en maquina virtual debian, repo bitbucket imagen de app en docker.]]