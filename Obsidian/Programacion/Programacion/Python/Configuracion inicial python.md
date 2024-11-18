
Stack 
#fastapi
#sqlalchemy
#alembic


estructura 
myproject/
├── app/
│   ├── controllers/
│   ├── models/
│   ├── routes/
│   │   └── __init__.py
│   ├── schema/
│   ├── services/
│   ├── middlewares/
│   └── main.py
├── config/
│   ├── __init__.py
│   └── database.py
├── database/
│   ├── migrations/
│   │   └── __init__.py
│   ├── factories/
│   │   └── __init__.py
│   └── seeders/
│       └── __init__.py
├── tests/
└── test_db_connection.py

en la raiz 
alembic init alembic

test db connection:
from sqlalchemy import create_engine, text

  

DATABASE_URL = "mysql://root@localhost/pecerapy"

engine = create_engine(DATABASE_URL)

  

with engine.connect() as connection:

    result = connection.execute(text("SELECT 1"))

    print(result.fetchone()) 


config data base 

from sqlalchemy import create_engine

  

DATABASE_URL = "mysql://root@localhost:3306/pecerapy"

  

engine = create_engine(DATABASE_URL)

main py :

from fastapi import FastAPI

  

from app.routes import api_router

  

app = FastAPI()

  

app.include_router(api_router)


model language
# app/models/language.py

  

from sqlalchemy import Column, Integer, String, DateTime

from sqlalchemy.sql import func

from app.models import Base

  

class Language(Base):

    __tablename__ = 'db_languages'

    id = Column(Integer, primary_key=True)

    name = Column(String(255), nullable=True)

    abbreviation = Column(String(10), nullable=True)

    flag = Column(String(255), nullable=True)

    order = Column(Integer, nullable=True)

    created_at = Column(DateTime, default=func.now(), nullable=False)

    updated_at = Column(DateTime, default=func.now(), onupdate=func.now(), nullable=False)

    deleted_at = Column(DateTime, nullable=True)  # Columna para Soft Delete

  

    def soft_delete(self):

        """Marca el registro como eliminado."""

        self.deleted_at = func.now()

        session.commit()

  

    def restore(self):

        """Restaura el registro eliminado."""

        self.deleted_at = None

        session.commit()

  

    @classmethod

    def query_active(cls):

        """Retorna solo los registros que no han sido eliminados."""

        return session.query(cls).filter(cls.deleted_at.is_(None))
### Crear una Migración para la Tabla

Necesitamos crear una migración para definir la estructura de la tabla en la base de datos.

sh

Copiar código

`alembic revision --autogenerate -m "create db_languages table with soft delete"`



path 

$env:PYTHONPATH = "$env:PYTHONPATH;C:\Users\abuse\Desktop\repositorio\VisualCode\python"



seeder de ejemplo 

from sqlalchemy import create_engine

from sqlalchemy.orm import sessionmaker

from app.models.language import Language

from app.models import Base

  

# Configuración de la base de datos

DATABASE_URL = "mysql://root@localhost/pecerapy"

engine = create_engine(DATABASE_URL)

SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

  

def seed_languages():

    session = SessionLocal()

    try:

        # Crear los datos a insertar

        new_language = Language(

            id=3,  # Asegúrate de que el ID no esté en uso

            name="french",

            abbreviation="FR",

            flag="french.gif",

            order=98

        )

  

        # Añadir el registro a la base de datos

        session.add(new_language)

        session.commit()

        print("Inserted French language successfully.")

    except Exception as e:

        print(f"An error occurred: {e}")

        session.rollback()

    finally:

        session.close()

  

if __name__ == "__main__":

    seed_languages()
revision alembic crear tabla 
alembic revision --autogenerate -m "create db_languages table with soft delete"


correr migraciones en python 

alembic upgrade head

[[Python]]










   
