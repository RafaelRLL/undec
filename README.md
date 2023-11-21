# undec
import os
import struct


def crear_archivo():
    if os.path.exists("pacientesHospital.dat"):
        respuesta = input("El archivo ya existe. ¿Desea crearlo nuevamente? (S/N): ")
        if respuesta.lower() != 's':
            return
    with open("pacientesHospital.dat", "wb"):
        print("Archivo creado con éxito.")


def cargar_registros():
    with open("pacientesHospital.dat", "ab") as archivo:
        while True:
            nroHC = int(input("Número de Historia Clínica (0 para finalizar): "))
            if nroHC == 0:
                break
            nomApellido = input("Nombre y Apellido: ")
            dni = int(input("DNI: "))
            edad = int(input("Edad: "))
            genero = input("Género: ")
            condicion = int(input("Condición (1-5): "))

            escribir_registro(archivo, nroHC, nomApellido, dni, edad, genero, condicion)

    print("Registros cargados con éxito.")

def escribir_registro(archivo, nroHC, nomApellido, dni, edad, genero, condicion):
    # Formatear el nombre y apellido a 20 caracteres, recortando o rellenando
    nomApellido = nomApellido[:20].ljust(20)

    # Convertir el género a bytes (utf-8)
    genero_bytes = genero.encode('utf-8')

    # Escribir los datos en el archivo
    archivo.write(nroHC.to_bytes(4, 'little') + nomApellido.encode('utf-8') + dni.to_bytes(4, 'little') +
                  edad.to_bytes(4, 'little') + genero_bytes + condicion.to_bytes(4, 'little'))

# Uso:
with open("pacientesHospital.dat", "ab") as archivo:
    nroHC = 123
    nomApellido = "Nombre Apellido"
    dni = 45678901
    edad = 30
    genero = "F"
    condicion = 3

    escribir_registro(archivo, nroHC, nomApellido, dni, edad, genero, condicion)
def consultar_registro():
    dni_buscar = int(input("Ingrese el DNI del paciente a consultar: "))
    with open("pacientesHospital.dat", "rb") as archivo:
        while True:
            registro = archivo.read(struct.calcsize("i20si1i"))
            if not registro:
                break
            nroHC, nomApellido, dni, edad, genero, condicion = struct.unpack("i20si1i", registro)
            if dni == dni_buscar:
                print(
                    f"Nro. HC: {nroHC}, Nombre y Apellido: {nomApellido.decode('utf-8')}, DNI: {dni}, Edad: {edad}, Género: {genero.decode('utf-8')}, Condición: {condicion}")
                return
    print("Paciente no encontrado.")


def listar_pacientes():
    with open("pacientesHospital.dat", "rb") as archivo:
        print(
            "{:<10} {:<30} {:<15} {:<10} {:<10} {:<10}".format("Nro. HC", "Nombre y Apellido", "DNI", "Edad", "Género",
                                                               "Condición"))
        while True:
            registro = archivo.read(struct.calcsize("i20si1i"))
            if not registro:
                break
            nroHC, nomApellido, dni, edad, genero, condicion = struct.unpack("i20si1i", registro)
            print("{:<10} {:<30} {:<15} {:<10} {:<10} {:<10}".format(nroHC, nomApellido.decode('utf-8'), dni, edad,
                                                                     genero.decode('utf-8'), condicion))


def modificar_registro():
    dni_modificar = int(input("Ingrese el DNI del paciente a modificar: "))
    with open("pacientesHospital.dat", "r+b") as archivo:
        while True:
            posicion = archivo.tell()
            registro = archivo.read(struct.calcsize("i20si1i"))
            if not registro:
                break
            nroHC, nomApellido, dni, edad, genero, condicion = struct.unpack("i20si1i", registro)
            if dni == dni_modificar:
                print(
                    f"Registro actual: {nroHC}, {nomApellido.decode('utf-8')}, {dni}, {edad}, {genero.decode('utf-8')}, {condicion}")
                genero_nuevo = input("Nuevo género: ")
                edad_nueva = int(input("Nueva edad: "))
                condicion_nueva = int(input("Nueva condición (1-5): "))

                archivo.seek(posicion)
                archivo.write(
                    struct.pack("i20si1i", nroHC, nomApellido, dni, edad_nueva, genero_nuevo, condicion_nueva))
                print("Registro modificado con éxito.")
                return
    print("Paciente no encontrado.")


def borrar_registro():
    dni_eliminar = int(input("Ingrese el DNI del paciente a eliminar: "))
    with open("pacientesHospital.dat", "r+b") as archivo:
        while True:
            posicion = archivo.tell()
            registro = archivo.read(struct.calcsize("i20si1i"))
            if not registro:
                break
            nroHC, nomApellido, dni, edad, genero, condicion = struct.unpack("i20si1i", registro)
            if dni == dni_eliminar:
                print(
                    f"Registro a eliminar: {nroHC}, {nomApellido.decode('utf-8')}, {dni}, {edad}, {genero.decode('utf-8')}, {condicion}")
                confirmacion = input("¿Está seguro que desea eliminar este registro? (S/N): ")
                if confirmacion.lower() == 's':
                    archivo.seek(posicion)
                    archivo.write(b'\x00' * struct.calcsize("i20si1i"))
                    print("Registro eliminado con éxito.")
                return
    print("Paciente no encontrado.")


def main():
    while True:
        print("\nMenú:")
        print("1. Crear el archivo")
        print("2. Cargar registros de pacientes")
        print("3. Consultar registro de paciente")
        print("4. Listado de pacientes en formato tabla")
        print("5. Modificar registro de paciente")
        print("6. Borrar registro de paciente")
        print("7. Salir")

        opcion = int(input("Ingrese la opción deseada: "))

        if opcion == 1:
            crear_archivo()
        elif opcion == 2:
            cargar_registros()
        elif opcion == 3:
            consultar_registro()
        elif opcion == 4:
            listar_pacientes()
        elif opcion == 5:
            modificar_registro()
        elif opcion == 6:
            borrar_registro()
        elif opcion == 7:
            print("¡Hasta luego!")
            break
        else:
            print("Opción no válida. Intente nuevamente.")


if __name__ == "__main__":
    main()
