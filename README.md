# my-budget
une application où tu peux gérer et contrôler votre budget 
from kivy.app import App
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.label import Label
from kivy.uix.button import Button
from kivy.uix.textinput import TextInput
from kivy.uix.popup import Popup
from kivy.uix.spinner import Spinner
from kivy.uix.gridlayout import GridLayout
from kivy.uix.floatlayout import FloatLayout
from kivy.uix.image import Image
from kivy.uix.scrollview import ScrollView
from kivy.core.window import Window

# Set window size for better appearance
Window.size = (400, 600)

# Définir la classe Budget
class Budget:
    categories_depenses = (
        "Logement", "Alimentation", "Transport", "Santé", "Éducation",
        "Internet", "Habillement", "Épargne et dettes", "Loisirs",
        "Assurances", "Énergie", "Équipements ménagers", "Frais bancaires", "Impôts et taxes"
    )

    def __init__(self, nom, budget):
        self.nom = nom
        self.budget = budget
        self.depenses = []
        self.total_par_categorie = {categorie: 0 for categorie in self.categories_depenses}

    def ajouter_depense(self, categorie, montant):
        if montant > self.budget:
            return "Dépassement du budget !"
        if montant > 500:
            return f"Alerte : {montant} MAD dépasse le seuil de 500 MAD."
        
        self.depenses.append({"categorie": categorie, "montant": montant})
        self.total_par_categorie[categorie] += montant
        self.budget -= montant
        return "Dépense ajoutée avec succès !"

    def get_totaux(self):
        return self.total_par_categorie

    def get_rapport(self):
        rapport = f"Nom : {self.nom}\nBudget restant : {self.budget} MAD\n\nDétail des dépenses :\n"
        for depense in self.depenses:
            rapport += f"- {depense['categorie']} : {depense['montant']} MAD\n"
        return rapport


# Classe principale de l'interface Kivy
class BudgetApp(App):
    def build(self):
        self.budget = None

        # Root layout with background image
        self.root_layout = FloatLayout()

        # Add background image
        self.background = Image(
            source='moneymanagement.jpg',  # Assurez-vous que l'image est dans le bon répertoire
            allow_stretch=True,
            keep_ratio=False,
            size_hint=(1, 1)
        )
        self.root_layout.add_widget(self.background)

        # Main layout for widgets
        self.layout = BoxLayout(orientation='vertical', padding=20, spacing=10, size_hint=(0.9, 0.9), pos_hint={'center_x': 0.5, 'center_y': 0.5})
        self.root_layout.add_widget(self.layout)

        # Title label
        self.title_label = Label(
            text="Gestion de Budget",
            font_size=24,
            bold=True,
            color=(1, 1, 1, 1)  # White color
        )
        self.layout.add_widget(self.title_label)

        # Input fields
        self.nom_entry = TextInput(
            hint_text="Entrez votre nom",
            multiline=False,
            size_hint=(1, None),
            height=40,
            background_color=(1, 1, 1, 0.8),  # Semi-transparent white
            foreground_color=(0, 0, 0, 1)
        )
        self.budget_entry = TextInput(
            hint_text="Budget initial (MAD)",
            multiline=False,
            size_hint=(1, None),
            height=40,
            background_color=(1, 1, 1, 0.8),  # Semi-transparent white
            foreground_color=(0, 0, 0, 1)
        )

        # Start button
        self.start_button = Button(
            text="Démarrer",
            size_hint=(1, None),
            height=50,
            background_color=(0.2, 0.6, 1, 1),  # Blue color
            color=(1, 1, 1, 1)  # White text
        )
        self.start_button.bind(on_press=self.init_budget)

        # Add widgets to layout
        self.layout.add_widget(self.nom_entry)
        self.layout.add_widget(self.budget_entry)
        self.layout.add_widget(self.start_button)

        return self.root_layout

    def init_budget(self, instance):
        try:
            nom = self.nom_entry.text
            budget = float(self.budget_entry.text)
            if budget <= 0:
                self.show_popup("Erreur", "Le budget doit être un nombre positif.")
                return
            self.budget = Budget(nom, budget)
            self.menu_principal()
        except ValueError:
            self.show_popup("Erreur", "Veuillez entrer un budget valide.")

    def show_popup(self, title, message):
        popup = Popup(
            title=title,
            content=Label(text=message, font_size=16),
            size_hint=(None, None),
            size=(400, 200)
        )
        popup.open()

    def menu_principal(self):
        self.layout.clear_widgets()

        # Main menu layout
        menu_layout = GridLayout(cols=1, padding=20, spacing=10, size_hint=(0.9, 0.9), pos_hint={'center_x': 0.5, 'center_y': 0.5})

        # Category spinner
        self.categorie_menu = Spinner(
            text="Choisissez une catégorie",
            values=Budget.categories_depenses,
            size_hint=(1, None),
            height=40,
            background_color=(1, 1, 1, 0.8),  # Semi-transparent white
            color=(0, 0, 0, 1)
        )

        # Amount input
        self.montant_entry = TextInput(
            hint_text="Montant (MAD)",
            multiline=False,
            size_hint=(1, None),
            height=40,
            background_color=(1, 1, 1, 0.8),  # Semi-transparent white
            foreground_color=(0, 0, 0, 1)
        )

        # Buttons
        self.ajouter_button = Button(
            text="Ajouter Dépense",
            size_hint=(1, None),
            height=50,
            background_color=(0.2, 0.8, 0.2, 1),  # Green color
            color=(1, 1, 1, 1)  # White text
        )
        self.ajouter_button.bind(on_press=self.ajouter_depense)

        self.totaux_button = Button(
            text="Voir Totaux",
            size_hint=(1, None),
            height=50,
            background_color=(0.8, 0.2, 0.2, 1),  # Red color
            color=(1, 1, 1, 1)  # White text
        )
        self.totaux_button.bind(on_press=self.afficher_totaux)

        self.quit_button = Button(
            text="Quitter",
            size_hint=(1, None),
            height=50,
            background_color=(0.5, 0.5, 0.5, 1),  # Gray color
            color=(1, 1, 1, 1)  # White text
        )
        self.quit_button.bind(on_press=self.stop)

        # Add widgets to menu layout
        menu_layout.add_widget(self.categorie_menu)
        menu_layout.add_widget(self.montant_entry)
        menu_layout.add_widget(self.ajouter_button)
        menu_layout.add_widget(self.totaux_button)
        menu_layout.add_widget(self.quit_button)

        self.layout.add_widget(menu_layout)

    def ajouter_depense(self, instance):
        try:
            categorie = self.categorie_menu.text
            montant = float(self.montant_entry.text)
            if montant <= 0:
                self.show_popup("Erreur", "Le montant doit être un nombre positif.")
                return
            message = self.budget.ajouter_depense(categorie, montant)
            self.show_popup("Réponse", message)
        except ValueError:
            self.show_popup("Erreur", "Veuillez entrer un montant valide.")

    def afficher_totaux(self, instance):
        totaux = self.budget.get_totaux()
        message = "Total des dépenses par catégorie :\n"
        
        # Prepare the formatted message for the popup content
        for categorie, total in totaux.items():
            message += f"{categorie}: {total} MAD\n"

        # Create a ScrollView to display the totals
        scroll_view = ScrollView(size_hint=(1, None), size=(400, 400))  # Adjust size for content
        content = Label(
            text=message,
            font_size=16,
            size_hint_y=None,  # Ensures the label grows with content
            text_size=(380, None)  # Ensure the text wraps correctly inside the scrollview
        )
        content.bind(texture_size=content.setter('size'))  # Adjust the label size based on content

        # Add the content to the ScrollView
        scroll_view.add_widget(content)

        # Create a Popup to display the totals
        popup = Popup(
            title="Totaux des Dépenses",
            content=scroll_view,
            size_hint=(None, None),
            size=(450, 450)  # Adjust this size if necessary to fit all content
        )
        popup.open()
        
    if __name__ == "__main__":
    BudgetApp().run()



