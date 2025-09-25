import tkinter as tk
from tkinter import ttk, messagebox
from datetime import datetime

# --- Constantes de estilo ---
FUENTE = ("Times New Roman", 10)
COLOR_FONDO_APP = "#b22222"  
COLOR_ENTRADA_FONDO = "#000000"  
COLOR_ENTRADA_TEXTO = "white"

# --- Clases base ---
class Cliente:
    def __init__(self, nombre, ap_paterno, ap_materno, nacimiento, domicilio):
        self.nombre = nombre
        self.ap_paterno = ap_paterno
        self.ap_materno = ap_materno
        self.nacimiento = nacimiento
        self.domicilio = domicilio

class Cuenta:
    def __init__(self, numero):
        self.numero = numero
        self.saldo = 1000.0

    def abonar(self, cantidad):
        self.saldo += cantidad

    def cargar(self, cantidad):
        if self.saldo >= cantidad:
            self.saldo -= cantidad
            return True
        return False

class Movimiento:
    def __init__(self, descripcion, tipo, cantidad, saldo):
        self.fecha = datetime.now().strftime("%Y-%m-%d")
        self.descripcion = descripcion
        self.tipo = tipo
        self.cantidad = cantidad
        self.saldo = saldo

class EstadoCuenta:
    def __init__(self, cliente, cuenta):
        self.fecha_ingreso = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        self.cliente = cliente
        self.cuenta = cuenta
        self.movimientos = []

    def agregar_movimiento(self, descripcion, tipo, cantidad):
        if tipo == "cargo" and not self.cuenta.cargar(cantidad):
            return False
        if tipo == "abono":
            self.cuenta.abonar(cantidad)

        self.movimientos.append(Movimiento(descripcion, tipo, cantidad, self.cuenta.saldo))
        return True

    def total_abonos(self):
        return sum(m.cantidad for m in self.movimientos if m.tipo == "abono")

    def total_cargos(self):
        return sum(m.cantidad for m in self.movimientos if m.tipo == "cargo")


class EdoCuentaApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Estado de Cuenta Bancario")
        self.root.configure(bg=COLOR_FONDO_APP)
        self.estado_cuenta = None

        self._crear_widgets_cliente()
        self._crear_widgets_movimiento()
        self._crear_widgets_resumen()

    def _crear_widgets_cliente(self):
        frame = tk.LabelFrame(self.root, text="Datos del Cliente / Cuenta", bg=COLOR_FONDO_APP, fg="white", font=FUENTE)
        frame.grid(row=0, column=0, padx=10, pady=10, sticky="ew")

        self.var_nombre = tk.StringVar()
        self.var_apellido_p = tk.StringVar()
        self.var_apellido_m = tk.StringVar()
        self.var_fecha_nac = tk.StringVar()
        self.var_domicilio = tk.StringVar()
        self.var_num_cuenta = tk.StringVar()

        campos = [
            ("Nombre:", self.var_nombre),
            ("Apellido Paterno:", self.var_apellido_p),
            ("Apellido Materno:", self.var_apellido_m),
            ("Fecha Nacimiento (YYYY-MM-DD):", self.var_fecha_nac),
            ("Domicilio:", self.var_domicilio),
            ("Número de cuenta:", self.var_num_cuenta),
        ]

        for i, (label_text, var) in enumerate(campos):
            tk.Label(frame, text=label_text, font=FUENTE, bg=COLOR_FONDO_APP, fg="white").grid(row=i, column=0, sticky="w", padx=5, pady=2)
            tk.Entry(frame, textvariable=var, font=FUENTE, bg=COLOR_ENTRADA_FONDO, fg=COLOR_ENTRADA_TEXTO).grid(row=i, column=1, sticky="ew", padx=5, pady=2)

        frame.columnconfigure(1, weight=1)

        boton = tk.Button(frame, text="Crear Cuenta", font=FUENTE, bg="white", command=self.crear_cuenta)
        boton.grid(row=len(campos), column=1, pady=5, sticky="e", padx=5)

    def _crear_widgets_movimiento(self):
        frame = tk.LabelFrame(self.root, text="Agregar Movimiento", bg=COLOR_FONDO_APP, fg="white", font=FUENTE)
        frame.grid(row=1, column=0, padx=10, pady=10, sticky="ew")

        self.var_descripcion = tk.StringVar()
        self.var_tipo = tk.StringVar(value="abono")
        self.var_monto = tk.StringVar()

        tk.Label(frame, text="Descripción:", font=FUENTE, bg=COLOR_FONDO_APP, fg="white").grid(row=0, column=0, sticky="w", padx=5, pady=2)
        tk.Entry(frame, textvariable=self.var_descripcion, font=FUENTE, bg=COLOR_ENTRADA_FONDO, fg=COLOR_ENTRADA_TEXTO).grid(row=0, column=1, sticky="ew", padx=5, pady=2)

        tk.Label(frame, text="Tipo:", font=FUENTE, bg=COLOR_FONDO_APP, fg="white").grid(row=0, column=2, sticky="w", padx=5, pady=2)
        combo = ttk.Combobox(frame, textvariable=self.var_tipo, values=["abono", "cargo"], state="readonly", font=FUENTE)
        combo.grid(row=0, column=3, sticky="ew", padx=5, pady=2)

        tk.Label(frame, text="Monto:", font=FUENTE, bg=COLOR_FONDO_APP, fg="white").grid(row=0, column=4, sticky="w", padx=5, pady=2)
        tk.Entry(frame, textvariable=self.var_monto, font=FUENTE, bg=COLOR_ENTRADA_FONDO, fg=COLOR_ENTRADA_TEXTO).grid(row=0, column=5, sticky="ew", padx=5, pady=2)

        frame.columnconfigure(1, weight=2)
        frame.columnconfigure(3, weight=1)
        frame.columnconfigure(5, weight=1)

        boton = tk.Button(frame, text="Agregar Movimiento", font=FUENTE, bg="white", command=self.agregar_movimiento)
        boton.grid(row=0, column=6, padx=10, pady=2)

    def _crear_widgets_resumen(self):
        frame = tk.LabelFrame(self.root, text="Estado de Cuenta - Movimientos", bg=COLOR_FONDO_APP, fg="white", font=FUENTE)
        frame.grid(row=2, column=0, padx=10, pady=10, sticky="nsew")

        cols = ["Fecha", "Descripción", "Tipo", "Cantidad", "Saldo"]
        self.tabla = ttk.Treeview(frame, columns=cols, show="headings", height=10)
        for col in cols:
            self.tabla.heading(col, text=col)
            self.tabla.column(col, width=100, anchor="center")
        self.tabla.pack(fill="both", expand=True, padx=5, pady=5)

        self.totales_label = tk.Label(frame, font=(FUENTE[0], 10, "bold"), bg=COLOR_FONDO_APP, fg="white", justify="left")
        self.totales_label.pack(anchor="w", padx=5, pady=5)

    
        self.root.rowconfigure(2, weight=1)
        self.root.columnconfigure(0, weight=1)

    def crear_cuenta(self):
        if not (self.var_nombre.get() and self.var_apellido_p.get() and self.var_apellido_m.get() and
                self.var_fecha_nac.get() and self.var_domicilio.get() and self.var_num_cuenta.get()):
            messagebox.showwarning("Datos faltantes", "Por favor completa todos los datos del cliente y la cuenta.")
            return

        try:
            # Validar fecha formato simple
            datetime.strptime(self.var_fecha_nac.get(), "%Y-%m-%d")

            cliente = Cliente(
                self.var_nombre.get(),
                self.var_apellido_p.get(),
                self.var_apellido_m.get(),
                self.var_fecha_nac.get(),
                self.var_domicilio.get()
            )
            cuenta = Cuenta(self.var_num_cuenta.get())
            self.estado_cuenta = EstadoCuenta(cliente, cuenta)
            messagebox.showinfo("Cuenta creada", "La cuenta ha sido creada con saldo inicial de $1000.00")
            self._actualizar_tabla()
        except ValueError:
            messagebox.showerror("Error", "Formato de fecha inválido. Use YYYY-MM-DD.")

    def agregar_movimiento(self):
        if self.estado_cuenta is None:
            messagebox.showwarning("Cuenta no creada", "Primero debes crear la cuenta.")
            return

        desc = self.var_descripcion.get().strip()
        tipo = self.var_tipo.get()
        try:
            cantidad = float(self.var_monto.get())
            if cantidad <= 0:
                raise ValueError
        except ValueError:
            messagebox.showerror("Error", "Cantidad inválida")
            return

        if not desc:
            messagebox.showerror("Error", "La descripción no puede estar vacía")
            return

        exito = self.estado_cuenta.agregar_movimiento(desc, tipo, cantidad)
        if not exito:
            messagebox.showwarning("Saldo insuficiente", "No hay saldo suficiente para el cargo.")
            return

        self.var_descripcion.set("")
        self.var_monto.set("")
        self._actualizar_tabla()

    def _actualizar_tabla(self):
        if self.estado_cuenta is None:
            return
        # Limpiar tabla
        for i in self.tabla.get_children():
            self.tabla.delete(i)
        # Insertar movimientos
        for m in self.estado_cuenta.movimientos:
            self.tabla.insert("", "end", values=(
                m.fecha,
                m.descripcion,
                m.tipo.capitalize(),
                f"${m.cantidad:.2f}",
                f"${m.saldo:.2f}"
            ))
        # Actualizar totales
        totales = f"Total Abonos: ${self.estado_cuenta.total_abonos():.2f}   |   Total Cargos: ${self.estado_cuenta.total_cargos():.2f}   |   Saldo Final: ${self.estado_cuenta.cuenta.saldo:.2f}"
        self.totales_label.config(text=totales)


if __name__ == "__main__":
    root = tk.Tk()
    app = EdoCuentaApp(root)
    root.mainloop()
