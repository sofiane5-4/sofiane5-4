import streamlit as st
import pandas as pd
import folium
from streamlit_folium import folium_static

# Fonction pour charger les données à partir du fichier CSV
def load_data(file_path):
    return pd.read_csv(file_path)

# Fonction pour sauvegarder les données dans le fichier CSV
def save_data(file_path, data):
    data.to_csv(file_path, index=False)

# Chemin vers le fichier CSV
file_path = 'clients.csv'

# Charger les données
data = load_data(file_path)

# Interface utilisateur pour afficher les informations du client
def afficher_informations(client):
    st.image(client['logo'])
    st.write(f"**Nom :** {client['nom']}")
    st.write(f"**Site web :** {client['site_web']}")
    st.write(f"**Activité :** {client['activité']}")
    st.write(f"**Chargé d'affaires :** {client['chargé_affaires']}")
    st.write(f"**Activités proposées :** {client['activités_proposées']}")
    if not pd.isna(client['concurrent']):
        st.write(f"**Concurrent :** {client['concurrent']}")
    if not pd.isna(client['date_echeance']):
        st.write(f"**Date d'échéance du contrat :** {client['date_echeance']}")

    # Afficher la carte
    map = folium.Map(location=[client['latitude'], client['longitude']], zoom_start=13)
    folium.Marker([client['latitude'], client['longitude']], tooltip=client['nom']).add_to(map)
    folium_static(map)

# Interface utilisateur principale
st.title("Tableau de bord interactif")
ville = st.selectbox('Sélectionnez une ville', data['ville'].unique())
clients = data[data['ville'] == ville]
client_nom = st.selectbox('Sélectionnez un client', clients['nom'].unique())

if client_nom:
    client_info = clients[clients['nom'] == client_nom].iloc[0]
    afficher_informations(client_info)

# Interface utilisateur pour ajouter un nouveau client
st.header("Ajouter un nouveau client")
with st.form(key='new_client_form'):
    new_ville = st.text_input('Ville')
    new_nom = st.text_input('Nom du client')
    new_logo = st.text_input('URL du logo')
    new_site_web = st.text_input('Site web')
    new_activité = st.text_input('Activité')
    new_chargé_affaires = st.text_input('Chargé d\'affaires')
    new_activités_proposées = st.text_input('Activités proposées')
    new_concurrent = st.text_input('Concurrent')
    new_date_echeance = st.date_input('Date d\'échéance du contrat')
    new_latitude = st.number_input('Latitude', format="%.6f")
    new_longitude = st.number_input('Longitude', format="%.6f")
    
    submit_button = st.form_submit_button(label='Ajouter')

    if submit_button:
        new_data = pd.DataFrame({
            'ville': [new_ville],
            'nom': [new_nom],
            'logo': [new_logo],
            'site_web': [new_site_web],
            'activité': [new_activité],
            'chargé_affaires': [new_chargé_affaires],
            'activités_proposées': [new_activités_proposées],
            'concurrent': [new_concurrent],
            'date_echeance': [new_date_echeance],
            'latitude': [new_latitude],
            'longitude': [new_longitude]
        })
        data = pd.concat([data, new_data], ignore_index=True)
        save_data(file_path, data)
        st.success('Le nouveau client a été ajouté avec succès !')
