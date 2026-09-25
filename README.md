import os
import math
import pygame
from pygame.locals import *
from OpenGL.GL import *
from OpenGL.GLU import *

# ==========================================
# DEFINICIÓN DEL CUBO (Vértices y Aristas)
# ==========================================
# 8 Vértices del cubo unitario centrado
vertices = [
    [-1.0, -1.0, 1.0],
    [1.0, -1.0, 1.0],
    [1.0, 1.0, 1.0],
    [-1.0, 1.0, 1.0],
    [-1.0, -1.0, -1.0],
    [1.0, -1.0, -1.0],
    [1.0, 1.0, -1.0],
    [-1.0, 1.0, -1.0]
]

# 12 Aristas que conectan los vértices
aristas = [
    (0, 1), (1, 2), (2, 3), (3, 0),  # Cara frontal
    (4, 5), (5, 6), (6, 7), (7, 4),  # Cara trasera
    (0, 4), (1, 5), (2, 6), (3, 7)  # Conexiones entre caras
]


# ==========================================
# FUNCIONES DE MATRICES 3D (Matemáticas Puras)
# ==========================================
def obtener_matriz_traslacion(tx, ty, tz):
    return [
        [1.0, 0.0, 0.0, tx],
        [0.0, 1.0, 0.0, ty],
        [0.0, 0.0, 1.0, tz],
        [0.0, 0.0, 0.0, 1.0]
    ]


def obtener_matriz_escala(sx, sy, sz):
    return [
        [sx, 0.0, 0.0, 0.0],
        [0.0, sy, 0.0, 0.0],
        [0.0, 0.0, sz, 0.0],
        [0.0, 0.0, 0.0, 1.0]
    ]


def obtener_matriz_rotacion_y(angulo_grados):
    rad = math.radians(angulo_grados)
    c = math.cos(rad)
    s = math.sin(rad)
    return [
        [c, 0.0, s, 0.0],
        [0.0, 1.0, 0.0, 0.0],
        [-s, 0.0, c, 0.0],
        [0.0, 0.0, 0.0, 1.0]
    ]


# ==========================================
# LOGICA DE RENDERIZADO Y DIBUJO
# ==========================================
def dibujar_cubo():
    """Dibuja el cubo en modo alambre con líneas blancas"""
    glBegin(GL_LINES)
    glColor3f(1.0, 1.0, 1.0)  # Color blanco para las aristas
    for arista in aristas:
        for vertice_idx in arista:
            glVertex3fv(vertices[vertice_idx])
    glEnd()


def configurar_proyeccion(modo_perspectiva, ancho, alto):
    """Establece el modo entre perspectiva (gluPerspective) y ortográfica (glOrtho)"""
    glMatrixMode(GL_PROJECTION)
    glLoadIdentity()

    aspecto = ancho / alto if alto != 0 else 1.0

    if modo_perspectiva:
        # Modo Perspectiva
        gluPerspective(45, aspecto, 0.1, 50.0)
    else:
        # Modo Ortográfico (Ajustado proporcionalmente para evitar deformación)
        glOrtho(-3.0 * aspecto, 3.0 * aspecto, -3.0, 3.0, 0.1, 50.0)

    glMatrixMode(GL_MODELVIEW)


def ejecutar_visualizador_cubo():
    """Inicializa la ventana de PyGame y maneja el bucle gráfico"""
    pygame.init()
    ancho, alto = 800, 600
    pantalla = pygame.display.set_mode((ancho, alto), DOUBLEBUF | OPENGL)
    pygame.display.set_caption("Evaluacion 02 - Cubo 3D")

    # Configuración inicial de OpenGL
    glEnable(GL_DEPTH_TEST)
    glClearColor(0.0, 0.0, 0.0, 1.0)  # Fondo Negro

    # Variables de control
    en_ejecucion = True
    modo_perspectiva = True  # Inicia en perspectiva
    angulo_rotacion = 0.0

    # Posición inicial de la cámara / espacio
    camara_z = -6.0
    camara_x = 0.0
    camara_y = 0.0

    reloj = pygame.time.Clock()

    while en_ejecucion:
        # Captura de eventos del sistema y teclado
        for evento in pygame.event.get():
            if evento.type == pygame.QUIT:
                en_ejecucion = False
            elif evento.type == pygame.KEYDOWN:
                if evento.key == pygame.K_p:
                    modo_perspectiva = True
                    print("[INFO] Proyección cambiada a: PERSPECTIVA")
                elif evento.key == pygame.K_o:
                    modo_perspectiva = False
                    print("[INFO] Proyección cambiada a: ORTOGRÁFICA")
                # Permitir cambiar dinámicamente la posición de la cámara con flechas
                elif evento.key == pygame.K_UP:
                    camara_z += 0.5
                elif evento.key == pygame.K_DOWN:
                    camara_z -= 0.5

        # Configurar la proyección en cada frame
        configurar_proyeccion(modo_perspectiva, ancho, alto)

        # Limpiar buffers de pantalla y profundidad
        glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT)
        glLoadIdentity()

        # Aplicar posición de la cámara (Traslación de la escena)
        glTranslatef(camara_x, camara_y, camara_z)

        # Incrementar el ángulo para mantener una rotación continua en el eje Y
        angulo_rotacion += 1.0
        glRotatef(angulo_rotacion, 0.0, 1.0, 0.0)

        # Dibujar
        dibujar_cubo()

        # Intercambiar el buffer de dibujo
        pygame.display.flip()
        reloj.tick(60)  # Limitar a 60 FPS

    pygame.quit()


# ==========================================
# MENÚ TUI E INICIO DEL PROGRAMA
# ==========================================
if __name__ == "__main__":
    opcion = ""
    while opcion != "2":
        # Limpiar la pantalla de la terminal según el Sistema Operativo (Debian)
        os.system('clear' if os.name == 'posix' else 'cls')

        print("=========================================")
        print("    MENÚ PRINCIPAL - EVALUACIÓN 02       ")
        print("=========================================")
        print("1. Visualizar Cubo 3D Animado")
        print("2. Salir del programa")
        print("=========================================")

        opcion = input("Seleccione una opción (1-2): ").strip()

        if opcion == "1":
            print("\nAbriendo ventana gráfica...")
            print("Controles dentro de la ventana:")
            print("  - Tecla [P]: Proyección Perspectiva")
            print("  - Tecla [O]: Proyección Ortográfica")
            print("  - Flechas [ARRIBA/ABAJO]: Mover posición de cámara Z")
            print("Cierre la ventana emergente para volver al menú.")
            ejecutar_visualizador_cubo()
        elif opcion == "2":
            print("\nFinalizando programa. ¡Hasta luego!")
        else:
            print("\nOpción no válida. Presione ENTER para continuar...")
            input()
