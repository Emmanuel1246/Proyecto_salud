# Proyecto_salud
Predicción de lunares malignos a través de redes neuronales
# -*- coding: utf-8 -*-
"""Salud.ipynb

Automatically generated by Colaboratory.

Original file is located at
    https://colab.research.google.com/drive/1LoV8azpU2hPX3j06t-Yq47pleQBEIDng
"""

#conectar con google
from google.colab import drive
drive.mount('/content/drive')

import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
import os
from glob import glob
import seaborn as sns
from PIL import Image

np.random.seed(42)
from sklearn.metrics import confusion_matrix

import keras
from keras.utils.np_utils import to_categorical # used for converting labels to one-hot-encoding
from keras.models import Sequential
from keras.layers import Dense, Dropout, Flatten, Conv2D, MaxPool2D, BatchNormalization
from sklearn.model_selection import train_test_split
from scipy import stats
from sklearn.preprocessing import LabelEncoder

skin_df = pd.read_csv('/content/drive/MyDrive/Analitica III/SALUD/salud/HAM10000_metadata.csv')

SIZE=64

# label encoding para los tipos de lunares
le = LabelEncoder()
le.fit(skin_df['dx'])
LabelEncoder()
print(list(le.classes_))
 
skin_df['label'] = le.transform(skin_df["dx"]) 
print(skin_df.sample(10))

# Data distribution 
fig = plt.figure(figsize=(15,10))

ax1 = fig.add_subplot(221)
skin_df['dx'].value_counts().plot(kind='bar', ax=ax1)
ax1.set_ylabel('Conteo')
ax1.set_title('Tipo de célula');

ax2 = fig.add_subplot(222)
skin_df['sex'].value_counts().plot(kind='bar', ax=ax2)
ax2.set_ylabel('Conteo', size=15)
ax2.set_title('Sexo');

ax3 = fig.add_subplot(223)
skin_df['localization'].value_counts().plot(kind='bar')
ax3.set_ylabel('Counteo',size=12)
ax3.set_title('ubicación')


ax4 = fig.add_subplot(224)
sample_age = skin_df[pd.notnull(skin_df['age'])]
sns.distplot(sample_age['age'], fit=stats.norm, color='red');
ax4.set_title('Edad')

plt.tight_layout()
plt.show()

# Crear la nueva columna que indica si la lesión es maligna o benigna
skin_df['label2'] = skin_df['dx'].apply(lambda x: 1 if x in ['akiec', 'bcc', 'mel'] else 0)
skin_df=skin_df.drop(['label'], axis=1)
# Mostrar el dataframe resultante
skin_df

# Distribucion de los datos segun su clase
from sklearn.utils import resample
print(skin_df['label2'].value_counts())

#Balanceo de datos.

df_0 = skin_df[skin_df['label2'] == 0]
df_1 = skin_df[skin_df['label2'] == 1]

n_samples0=3000
n_samples1=1954
df_0_balanced = resample(df_0, replace=True, n_samples=n_samples0, random_state=42) 
df_1_balanced = resample(df_1, replace=True, n_samples=n_samples1, random_state=42) 


skin_df_balanced = pd.concat([df_0_balanced,df_1_balanced])

print(skin_df_balanced['label2'].value_counts())

skin_df_balanced

#guardar la tabla en formato csv
ruta = '/content/drive/MyDrive/Analitica III/SALUD/tablaimgg.csv'
skin_df_balanced.to_csv(ruta , index=False)



--------------------------------------------------------------------------------

#conectar con google
from google.colab import drive
drive.mount('/content/drive')

import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
import os
from glob import glob
import seaborn as sns
from PIL import Image

np.random.seed(42)
from sklearn.metrics import confusion_matrix

import keras
from keras.utils.np_utils import to_categorical # used for converting labels to one-hot-encoding
from keras.models import Sequential
from keras.layers import Dense, Dropout, Flatten, Conv2D, MaxPool2D, BatchNormalization
from sklearn.model_selection import train_test_split
from scipy import stats
from sklearn.preprocessing import LabelEncoder
from tensorflow.keras.optimizers import RMSprop
from keras.optimizers import RMSprop


########Paquetes para NN #########
import tensorflow as tf
from sklearn import metrics ### para analizar modelo
from sklearn.ensemble import RandomForestClassifier  ### para analizar modelo

#PREPROCESAMIENTO

skin_df_balanced = pd.read_csv('/content/drive/MyDrive/Analitica III/SALUD/tablaimgg.csv')

skin_df_balanced

SIZE=32
# Crear un diccionario con la ruta de la imagen para cada imagen_id
image_path = {os.path.splitext(os.path.basename(x))[0]: x
              for x in glob(os.path.join('/content/drive/MyDrive/Analitica III/SALUD/salud', '*', '*.jpg'))}

# Agregar la ruta de la imagen como una nueva columna a la tabla
skin_df_balanced['path'] = skin_df_balanced['image_id'].map(image_path.get)

# Cargar la imagen en forma de un array numpy y agregarlo como una nueva columna a la tabla
skin_df_balanced['image'] = skin_df_balanced['path'].map(lambda x: np.asarray(Image.open(x).resize((SIZE,SIZE))))

skin_df_balanced

#graficar
n_samples = 5  

# Plot
fig, m_axs = plt.subplots(7, n_samples, figsize = (4*n_samples, 3*7))
for n_axs, (type_name, type_rows) in zip(m_axs, 
                                         skin_df_balanced.sort_values(['dx']).groupby('dx')):
    n_axs[0].set_title(type_name)
    for c_ax, (_, c_row) in zip(n_axs, type_rows.sample(n_samples, random_state=1234).iterrows()):
        c_ax.imshow(c_row['image'])
        c_ax.axis('off')
        
        
#Convertir columna de imagenes a un array
X = np.asarray(skin_df_balanced['image'].tolist())
X = X/255. # Scale values to 0-1. You can also used standardscaler or other scaling methods.
y = np.asarray(skin_df_balanced['label2'].tolist())
#Y=skin_df_balanced['label'] #Assign label values to Y
#Y_cat = to_categorical(Y, num_classes=7) #Convert to categorical as this is a multiclass classification problem

X = np.asarray(X).astype(np.float32)
y = np.asarray(y).astype(np.float32)

X

y

# Dividir la tabla en características (X) y etiquetas (y)
#X = skin_df_balanced['image']
#y = skin_df_balanced['image']

# Dividir los datos en un conjunto de entrenamiento y uno de prueba
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)


print(X_test.shape,y_test.shape)


######  normalizar variables ######
X_train /=255 ### escalaro para que quede entre 0 y 1
X_test /=255


X_train
