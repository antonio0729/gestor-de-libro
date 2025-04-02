# gestor-de-libro
import tkinter as tk
import sqlite3

class Libro:
    def __init__(self, titulo, autor, anio_publicacion):        
        self.titulo = titulo
        self.autor = autor
        self.anio_publicacion = anio_publicacion  

class BaseDeDatos:
    def __init__(self, nombre_bd):
        self.con = sqlite3.connect(nombre_bd)
        self.cur = self.con.cursor()
        self.crear_tabla()

    def crear_tabla(self):
        self.cur.execute('''
            CREATE TABLE IF NOT EXISTS libros (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                titulo TEXT,
                autor TEXT,
                anio_publicacion INTEGER
            )
        ''')
        self.con.commit()

    def agregar_libro(self, libro):
        self.cur.execute('INSERT INTO libros (titulo, autor, anio_publicacion) VALUES (?, ?, ?)',
                         (libro.titulo, libro.autor, libro.anio_publicacion))
        self.con.commit()

    def obtener_libros(self):
        self.cur.execute('SELECT * FROM libros')
        return self.cur.fetchall()

    def eliminar_libro(self, libro_id):
        self.cur.execute('DELETE FROM libros WHERE id = ?', (libro_id,))
        self.con.commit()

class App:
    def __init__(self, ventana_principal):
        self.ventana_principal = ventana_principal
        self.ventana_principal.title("Gestor de Libros")
        self.bd = BaseDeDatos('catalogo_libros.db')

        # Widgets de entrada
        self.label_titulo = tk.Label(self.ventana_principal, text='Título')
        self.label_titulo.pack(pady=5)
        self.entry_titulo = tk.Entry(self.ventana_principal)
        self.entry_titulo.pack(pady=5)

        self.label_autor = tk.Label(self.ventana_principal, text='Autor')
        self.label_autor.pack(pady=5)
        self.entry_autor = tk.Entry(self.ventana_principal)
        self.entry_autor.pack(pady=5)

        self.label_anio = tk.Label(self.ventana_principal, text='Año de Publicación')
        self.label_anio.pack(pady=5)
        self.entry_anio = tk.Entry(self.ventana_principal)
        self.entry_anio.pack(pady=5)

        # Lista de libros
        self.listbox = tk.Listbox(self.ventana_principal, width=50)
        self.listbox.pack(pady=5)

        # Botones
        self.boton_guardar = tk.Button(self.ventana_principal, text='Guardar', command=self.agregar_libro)
        self.boton_guardar.pack(pady=5)

        self.boton_mostrar = tk.Button(self.ventana_principal, text='Mostrar', command=self.mostrar_libros)
        self.boton_mostrar.pack(pady=5)

        self.boton_eliminar = tk.Button(self.ventana_principal, text='Eliminar', command=self.eliminar_libro)
        self.boton_eliminar.pack(pady=5)

        self.boton_exportar = tk.Button(self.ventana_principal, text='Exportar a TXT', command=self.exportar_libros)
        self.boton_exportar.pack(pady=5)

    def agregar_libro(self):
        titulo = self.entry_titulo.get().strip()
        autor = self.entry_autor.get().strip()
        anio = self.entry_anio.get().strip()

        if titulo and autor and anio.isdigit():
            libro = Libro(titulo, autor, int(anio))
            self.bd.agregar_libro(libro)
            self.mostrar_libros()
            self.entry_titulo.delete(0, tk.END)
            self.entry_autor.delete(0, tk.END)
            self.entry_anio.delete(0, tk.END)
        else:
            print("Datos inválidos. Por favor, completa todos los campos correctamente.")

    def mostrar_libros(self):
        self.listbox.delete(0, tk.END)
        libros = self.bd.obtener_libros()
        for libro in libros:
            self.listbox.insert(tk.END, f"{libro[0]} | {libro[1]} | {libro[2]} | {libro[3]}")

    def eliminar_libro(self):
        seleccion = self.listbox.curselection()
        if seleccion:
            index = seleccion[0]
            registro = self.listbox.get(index)
            libro_id = int(registro.split(" | ")[0])  # Separa por el delimitador " | "
            self.bd.eliminar_libro(libro_id)
            self.mostrar_libros()
        else:
            print("No hay ningún libro seleccionado para eliminar.")

    def exportar_libros(self):
        libros = self.bd.obtener_libros()
        with open('libros.txt', 'w', encoding='utf-8') as archivo:
            for libro in libros:
                archivo.write(f"{libro[0]} | {libro[1]} | {libro[2]} | {libro[3]}\n")
        print("Exportación completada a 'libros.txt'.")

if __name__ == "__main__":
    mi_ventana = tk.Tk()
    app = App(mi_ventana)
    mi_ventana.mainloop()
    
