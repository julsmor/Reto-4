# Reto-4
Include the class exercise in the repo.
The restaurant revisted
Add setters and getters to all subclasses for menu item
Override calculate_total_price() according to the order composition (e.g if the order includes a main course apply some disccount on beverages)
Add the class Payment() following the class example. 

**Descripción del ejercicio:**
Se crearon varias clases como `MenuItem`, `Beverage`, `Appetizer`, `MainCourse`, `Order` y `Payment`, implementando **encapsulamiento** mediante atributos privados y métodos *getters* y *setters* para acceder y modificar los datos de manera controlada. Además, se **sobrescribió el método `get_total_price()`** en las subclases para ajustar los precios según el tipo de producto y las condiciones de la orden (por ejemplo, aplicar un **descuento del 20% en bebidas** cuando el pedido incluye un plato principal). También se añadió la clase `Payment`, que simula el proceso de pago tanto con **tarjeta** como en **efectivo**, calculando el cambio o validando el monto entregado. Finalmente, se probó el sistema creando un menú, una orden con varios productos y mostrando cómo se calculan los totales con y sin descuentos, demostrando una aplicación completa de herencia, encapsulamiento y polimorfismo en un contexto práctico.

```
from typing import List, Optional


class MenuItem:
    def __init__(self, name: str, price: float):
        self.__name = str(name)
        self.__price = float(price)

    # getters / setters
    def get_name(self) -> str:
        return self.__name

    def set_name(self, name: str) -> None:
        self.__name = str(name)

    def get_price(self) -> float:
        return self.__price

    def set_price(self, price: float) -> None:
        self.__price = float(price)

    # precio total potencialmente dependiente del contexto de la orden
    def get_total_price(self, order: Optional["Order"] = None) -> float:
        return self.__price

    def __repr__(self):
        return f"{self.__class__.__name__}({self.get_name()}, {self.get_price():.2f})"


class Beverage(MenuItem):
    def __init__(self, name: str, price: float, size: str):
        super().__init__(name, price)
        self.__size = str(size).lower()  # "pequeño", "mediano", "grande"

    # getters / setters
    def get_size(self) -> str:
        return self.__size

    def set_size(self, size: str) -> None:
        self.__size = str(size).lower()

    # Sobrescribimos get_total_price para aplicar:
    # - factor por tamaño
    # - descuento si la orden incluye un MainCourse (ejemplo: 20% off en bebidas)
    def get_total_price(self, order: Optional["Order"] = None) -> float:
        base = self.get_price()
        # factor por tamaño
        size_factor = {"pequeño": 0.8, "mediano": 1.0, "grande": 1.2}
        factor = size_factor.get(self.__size, 1.0)
        price = base * factor

        # descuento si la orden contiene al menos un MainCourse
        if order is not None and order.has_main_course():
            # ejemplo: 20% off en bebidas si hay plato principal
            price *= 0.8

        return price


class Appetizer(MenuItem):
    def __init__(self, name: str, price: float, is_vegan: bool):
        super().__init__(name, price)
        self.__is_vegan = bool(is_vegan)

    # getters / setters
    def get_is_vegan(self) -> bool:
        return self.__is_vegan

    def set_is_vegan(self, is_vegan: bool) -> None:
        self.__is_vegan = bool(is_vegan)

    # por ahora no sobrescribimos precio, pero podríamos (por ejemplo, recargo por ingredientes)
    def get_total_price(self, order: Optional["Order"] = None) -> float:
        return super().get_total_price(order)


class MainCourse(MenuItem):
    def __init__(self, name: str, price: float, calories: int):
        super().__init__(name, price)
        self.__calories = int(calories)

    # getters / setters
    def get_calories(self) -> int:
        return self.__calories

    def set_calories(self, calories: int) -> None:
        self.__calories = int(calories)

    # ejemplo: podríamos introducir un cargo extra para platos con calorías > X (no obligatorio)
    def get_total_price(self, order: Optional["Order"] = None) -> float:
        return super().get_total_price(order)


class Order:
    def __init__(self):
        self.__items: List[MenuItem] = []

    # getters / setters
    def get_items(self) -> List[MenuItem]:
        return list(self.__items)  # copia superficial

    def set_items(self, items: List[MenuItem]) -> None:
        if not all(isinstance(i, MenuItem) for i in items):
            raise TypeError("All elements must be MenuItem instances")
        self.__items = list(items)

    def add_item(self, item: MenuItem) -> None:
        if not isinstance(item, MenuItem):
            raise TypeError("item must be MenuItem")
        self.__items.append(item)

    def has_main_course(self) -> bool:
        return any(isinstance(i, MainCourse) for i in self.__items)

    # calcular el total usando get_total_price(order) para que cada item pueda conocer la orden
    def calculate_total(self) -> float:
        total = 0.0
        for item in self.__items:
            total += item.get_total_price(order=self)
        return total

    # Descuento por cantidad (regla previa)
    def apply_discount(self) -> float:
        total = self.calculate_total()
        if len(self.__items) >= 5:
            return total * 0.9  # 10% de descuento por 5 o más items
        return total

    def clear(self) -> None:
        self.__items.clear()

    def __repr__(self):
        items_str = ", ".join(repr(i) for i in self.__items)
        return f"Order({items_str})"


class Payment:
    """
    Payment simple:
    - method: "cash" or "card"
    - process_payment(amount, paid_amount) devuelve (success, change_or_msg)
    """

    def __init__(self, method: str):
        self.__method = str(method).lower()
        if self.__method not in ("cash", "card"):
            raise ValueError("method must be 'cash' or 'card'")

    def get_method(self) -> str:
        return self.__method

    def set_method(self, method: str) -> None:
        if method.lower() not in ("cash", "card"):
            raise ValueError("method must be 'cash' or 'card'")
        self.__method = method.lower()

    def process_payment(self, amount: float, paid: Optional[float] = None):
        """
        Para 'card' asumimos que la transacción se autoriza y no requiere paid.
        Para 'cash' se requiere monto entregado (paid) y devolvemos el cambio si hay.
        """
        amount = float(amount)
        if self.__method == "card":
           
            return True, "Tarjeta autorizada"
        else:
            if paid is None:
                return False, "Paid amount required for cash payments"
            paid = float(paid)
            if paid < amount:
                return False, f"Pago insuficiente. Faltan {amount - paid:.2f}"
            change = paid - amount
            return True, change


# Ejemplo de uso

if __name__ == "__main__":
    # menú
    menu = [
        Beverage("Jugo de mango", 6000.0, "mediano"),
        Beverage("Agua con gas", 4000.0, "pequeño"),
        Beverage("Limonada", 5000.0, "grande"),
        Appetizer("Arepitas con guacamole", 8000.0, True),
        Appetizer("Empanadas de carne", 7000.0, False),
        Appetizer("Patacones con hogao", 7500.0, True),
        MainCourse("Bandeja paisa", 18000.0, 1200),
        MainCourse("Ajiaco", 16000.0, 900),
        MainCourse("Arroz con pollo", 14000.0, 850),
        MainCourse("Pasta al pesto", 15000.0, 700),
    ]

    order = Order()
    order.add_item(menu[0])  # Jugo mediano
    order.add_item(menu[3])  # Arepitas (entrada)
    order.add_item(menu[6])  # Bandeja paisa (plato fuerte) -> activará descuento en bebidas
    order.add_item(menu[7])  # Ajiaco
    order.add_item(menu[9])  # Pasta al pesto

    print("Orden:", order)
    print("Items individuales y su precio (considerando descuentos por composición):")
    for it in order.get_items():
        print(f" - {it.get_name()} ({it.__class__.__name__}): {it.get_total_price(order):.2f}")

    total = order.calculate_total()
    print(f"\nTotal sin descuento por cantidad: {total:.2f}")

    total_con_descuento_cantidad = order.apply_discount()
    print(f"Total con descuento por cantidad (si aplica): {total_con_descuento_cantidad:.2f}")

    # Ejemplo de pago con tarjeta
    payment = Payment("card")
    ok, msg = payment.process_payment(total_con_descuento_cantidad)
    print("\nPago con tarjeta:", ok, msg)

    # Ejemplo de pago en efectivo
    cash_payment = Payment("cash")
    paid_amount = total_con_descuento_cantidad + 2000.0  # cliente entrega un billete extra
    ok2, result = cash_payment.process_payment(total_con_descuento_cantidad, paid=paid_amount)
    if ok2:
        print("Pago efectivo aceptado. Cambio:", result)
    else:
        print("Pago efectivo falló:", result)

    # Usando setters: cambiar tamaño de la primera bebida a grande
    bebida = order.get_items()[0]
    if isinstance(bebida, Beverage):
        print("\nAntes size:", bebida.get_size(), "precio:", bebida.get_price())
        bebida.set_size("grande")
        print("Después size:", bebida.get_size(), "precio ajustado en total:", bebida.get_total_price(order))
```
