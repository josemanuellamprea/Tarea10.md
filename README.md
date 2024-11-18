# Tarea10.md

~~~
#!/bin/bash
#Autor: José Manuel Lamprea Demski
#Descripción: Script para gestión de pool en kvm

# Función para mostrar los pools existentes
ver_pools() {
    echo "Pools existentes:"
    virsh pool-list --all
}

# Función para crear discos duros en un pool
crear_discos() {
    echo "¿Cuántos discos duros deseas crear?"
    read num_discos
    for ((i=1; i<=num_discos; i++))
    do
        echo "Introduce el nombre del disco duro $i:"
        read nombre_disco
        echo "Introduce el tamaño del disco (en GB):"
        read tamano_disco
        # Crear el disco duro en el pool
        qemu-img create -f qcow2 /var/lib/libvirt/images/$nombre_disco.qcow2 ${tamano_disco}G
        echo "Disco duro $nombre_disco creado con tamaño ${tamano_disco}GB."
    done
}

# Función para mostrar la información de los discos duros
informacion_discos() {
    echo "Introduce el nombre del disco duro que deseas inspeccionar:"
    read nombre_disco
    ls -lh /var/lib/libvirt/images/$nombre_disco.qcow2
}

# Función para agregar discos a una máquina virtual
agregar_disco() {
    echo "Introduce el nombre de la máquina virtual:"
    read nombre_vm
    echo "Introduce el nombre del disco duro a agregar:"
    read nombre_disco
    # Agregar el disco a la máquina virtual
    virsh attach-disk $nombre_vm /var/lib/libvirt/images/$nombre_disco.qcow2 vdb --persistent
    echo "Disco $nombre_disco agregado a la máquina virtual $nombre_vm."
}

# Función para crear, inicializar y configurar un pool con arranque automático
configurar_pool() {
    echo "Introduce el nombre del pool que deseas crear:"
    read nombre_pool
    echo "Introduce la ruta del directorio para el pool:"
    read ruta_pool
    # Crear el pool
    virsh pool-define-as $nombre_pool dir --target $ruta_pool
    virsh pool-build $nombre_pool
    virsh pool-start $nombre_pool
    virsh pool-autostart $nombre_pool
    echo "Pool $nombre_pool creado, inicializado y configurado con arranque automático."
}

# Función para borrar un pool
borrar_pool() {
    echo "Introduce el nombre del pool que deseas borrar:"
    read nombre_pool
    virsh pool-destroy $nombre_pool
    virsh pool-undefine $nombre_pool
    echo "Pool $nombre_pool borrado."
}

# Función para generar el informe
generar_informe() {
    pools_totales=$(virsh pool-list --all | wc -l)
    pools_activados=$(virsh pool-list --state-running | wc -l)
    pools_inactivos=$(virsh pool-list --state-inactive | wc -l)
    pools_auto=$(virsh pool-list --autostart | wc -l)

    echo "Informe de Pools:" > Informe_Pool.txt
    echo "Número total de pools creados: $((pools_totales-1))" >> Informe_Pool.txt
    echo "Número de pools activos: $((pools_activados-1))" >> Informe_Pool.txt
    echo "Número de pools inactivos: $((pools_inactivos-1))" >> Informe_Pool.txt
    echo "Número de pools con arranque automático: $((pools_auto-1))" >> Informe_Pool.txt
    echo "Informe generado en Informe_Pool.txt."
}

# Menú principal
while true; do
    echo "----- Menú de Gestión de Pools KVM -----"
    echo "1) Visualizar pools existentes"
    echo "2) Configurar un pool"
    echo "3) Borrar un pool"
    echo "4) Generar informe de pools"
    echo "5) Salir"
    read -p "Selecciona una opción: " opcion

    case $opcion in
        1)
            ver_pools
            ;;
        2)
            echo "Seleccione una sub-opción:"
            echo "a) Crear discos duros"
            echo "b) Mostrar información de discos duros"
            echo "c) Agregar discos a una máquina virtual"
            echo "d) Crear, inicializar y configurar un pool"
            read -p "Selecciona una sub-opción: " subopcion
            case $subopcion in
                a)
                    crear_discos
                    ;;
                b)
                    informacion_discos
                    ;;
                c)
                    agregar_disco
                    ;;
                d)
                    configurar_pool
                    ;;
                *)
                    echo "Opción no válida."
                    ;;
            esac
            ;;
        3)
            borrar_pool
            while true; do
                read -p "¿Deseas borrar otro pool? (s/n): " respuesta
                if [[ $respuesta == "n" ]]; then
                    break
                elif [[ $respuesta == "s" ]]; then
                    borrar_pool
                else
                    echo "Respuesta no válida."
                fi
            done
            ;;
        4)
            generar_informe
            ;;
        5)
            echo "Saliendo del programa..."
            break
            ;;
        *)
            echo "Opción no válida."
            ;;
    esac
done

~~~
