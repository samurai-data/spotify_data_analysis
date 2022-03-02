# Analyse de données Spotify


## Présentation du projet
Spotify est une plate-forme populaire de streaming musical développée en Suède. Du fait que cette plate-forme est pratiquement le seul biais par lequel j'écoute de la musique j'ai décidé de profiter de l'opportunité d'obtenir une copie de la plupart de mes données personnelles recueillies par ce service pour analyser mes habitudes musicales de l'année dernière (2021).

###### Plan
 
- Nettoyage des données obtenues
- Visualisation graphique
- Construction d'un tableau de bord dynamique
- Tentative d'une analyse de classification (clustering) par la méthode de k-means

###### Outils

- R Studio pour le traitement et l'analyse de données
- Tableau Public pour la construction du tableau de bord



## Chargement des données

```
setwd('C:/Users/Inessa/Desktop/Portfolio Projects/my_spotify_data/MyData')
streamHistory0 <- fromJSON("StreamingHistory0.json", flatten = TRUE)
streamHistory1 <- fromJSON("StreamingHistory1.json", flatten = TRUE)
streamHistory <- rbind(streamHistory0,streamHistory1)
myLibrary <- fromJSON("YourLibrary.json", flatten = TRUE)
myLibrary <- data.frame(myLibrary$tracks)
```

## Inspection visuelle des données

```
glimpse(streamHistory)
glimpse(myLibrary)
```
![Screenshot_1](https://user-images.githubusercontent.com/90149200/156365077-b758c0f7-5a85-44c2-bd68-b6f0dba2f26f.jpg)
![Screenshot_2](https://user-images.githubusercontent.com/90149200/156365108-7ba4ca8d-9dfb-4fea-9508-2c20a8027a4b.jpg)


Nombre des chansons uniques que j'ai écouté pendant un an du 7/01/2021 à 7/01/2022
 ```
songs <- data.frame(unique(streamHistory$trackName))
count(songs) #2309 chansons
artists <- data.frame(unique(streamHistory$artistName))
count(artists) #1151
rm(songs, artists)
```
2309 chansons et 1151 artistes uniques sont identifiés.

# Nettoyage

Fusionner les deux variables qualitatives pour créer une variable ID. Je modifie également les liens web qui permettent l'accès aux chansons. On s'en servira par la suite pour créer un tableau de bord sur un logiciel de visualisation des données Tableau.
```
myLibrary <- myLibrary %>% mutate(ID = paste(myLibrary$artist, myLibrary$track, sep = ':')) %>% 
  mutate(URL = "https://open.spotify.com/embed/track/") %>%
  mutate(id = sapply(uri, FUN = function(uri){substring(uri, regexpr('spotify:track:',uri) + 14, nchar(uri))}, USE.NAMES=FALSE) )
myLibrary <- mutate(URL = paste(URL, id, sep = ''), myLibrary)
glimpse(myLibrary)
```


```
#Problème: Date est un caractère
#On convertit alors cette variable en datetime
mySpotify <- streamHistory %>%
  mutate(endTime = as.POSIXct(endTime)) %>%
  mutate(date = floor_date(endTime, "day") %>% 
           as_date, weekday = wday(date,label = T, week_start = getOption("lubridate.week.start", 1)), seconds = msPlayed / 1000, minutes = seconds / 60, hours = minutes/60)
```

```
#Date avec l'heure arrondie
mySpotify <- mutate(date_hour = round_date(endTime,"hour"), mySpotify)
glimpse(mySpotify)

#Fusionner les deux bases
data <- merge(x = mySpotify, y = myLibrary[,c('ID', 'URL')], by = 'ID', all.x = T)
#Extraire l'heure arrondie uniquement à partir de la variable 'date_hour' dans la base fusionnée
data <- data %>% 
  mutate(hour = as.character(hour(date_hour))) %>% 
  mutate(hour = paste(hour, 'h' ))
glimpse(data)



data <- data %>% mutate(URL = ifelse(ID == 'Robbie Williams:Angels', 'https://open.spotify.com/embed/track/1M2nd8jNUkkwrc1dgBPTJz', 
                                     ifelse(ID == 'Sara Lov:Fountain', 'https://open.spotify.com/embed/track/12xDGs4NYCsoNKdHYnoZZr', 
                                          ifelse(ID == 'C. Tangana:Antes de Morirme (feat. ROSALÍA)', 'https://open.spotify.com/embed/track/4Dl8bEzhAEGbcwkQawS1XG', URL))))

```

```
#Pour jeter on coup d'oeuil sur la base finale
glimpse(data)
```

```
#La je reviens après la construction de mon tableau de bord sur Tableau afin de faire qqs ajustements
#Notamment rajouter manuellement les URL manquants nécessaires à la visualisation graphique
data <- data %>% mutate(URL = ifelse(ID == 'Robbie Williams:Angels', 'https://open.spotify.com/embed/track/1M2nd8jNUkkwrc1dgBPTJz', 
                                     ifelse(ID == 'Sara Lov:Fountain', 'https://open.spotify.com/embed/track/12xDGs4NYCsoNKdHYnoZZr', 
                                            ifelse(ID == 'C. Tangana:Antes de Morirme (feat. ROSALÍA)', 'https://open.spotify.com/embed/track/4Dl8bEzhAEGbcwkQawS1XG', URL))))


data <- data %>% mutate(artist_url = ifelse(artistName == 'The Lumineers', 'https://open.spotify.com/embed/artist/16oZKvXb6WkQlVAjwo2Wbg',
                                   ifelse(artistName == 'Mika Newton', 'https://open.spotify.com/embed/artist/74EZBv1sl8tSXbttKYhAC0',
                                          ifelse(artistName == 'Alai Oli', 'https://open.spotify.com/embed/artist/4snI0qikpQST1U1VWAxEY6',
                                                 ifelse(artistName == 'Robbie Williams', 'https://open.spotify.com/embed/artist/2HcwFjNelS49kFbfvMxQYw',
                                                        ifelse(artistName == 'Freakonomics Radio', '<https://open.spotify.com/embed/show/6z4NLXyHPga1UmSJsPK7G1',
                                                               ifelse(artistName == 'Amaral', 'https://open.spotify.com/embed/artist/4OkeTQCk0fvX6VBYpOOxDi',
                                                                      ifelse(artistName == 'Sara Lov', 'https://open.spotify.com/embed/artist/53qqj1ih4yfPRgUYtEJjVR',
                                                                             ifelse(artistName == 'Ed Sheeran', 'https://open.spotify.com/embed/artist/6eUKZXaKkcviH0Ku9w2n3V',
                                                                                    ifelse(artistName == 'Backstreet Boys', 'https://open.spotify.com/embed/artist/5rSXSAkZ67PYJSvpUpkOr7',
                                                                                           ifelse(artistName == 'Paramore', 'https://open.spotify.com/embed/artist/74XFHRwlV6OrjEM0A2NCMF', NA )))))))))))
```

```
#Exportation de la base pour Tableau
write.csv(data,"C:/Users/Inessa/Desktop/Portfolio Projects/my_spotify_data/mySpotify.csv")

```


# Construction des graphiques
```
# Playback hour 
dayHour <- data %>% 
  group_by(date, hour = hour(date_hour)) %>% 
  summarize(hoursListened = sum(hours))

```

```
#Graphique
dayHour %>% 
  ggplot(aes(x= hour, y= hoursListened, group = date)) +
  geom_col(fill = "darkcyan") +
  scale_fill_brewer(palette = 3) +
  scale_x_continuous(breaks = seq(0, 24, 2)) +
  scale_y_continuous(breaks = seq(0, 60, 5)) +
  labs(title = "Mon activité sur Spotify pendant une journée", subtitle = "Activité de 0 à 24 heures") +
  xlab("L'heure") +
  ylab("Nombre d'heures") +
  theme_gray()
#Activité la plus elevée entre 16h et 18h

```

## Clustering

###### Présentation des variables choisies pour l'analyse

- **Acousticness**: Une mesure de confiance de 0,0 à 1,0 indiquant si la piste est acoustique. 1.0 représente une confiance élevée que la piste est acoustique.
- **Danceability**: Danceability décrit à quel point une piste est appropriée pour la danse basée sur une combinaison d'éléments musicaux comprenant le tempo, la stabilité du rythme, la force du battement et la régularité générale. Une valeur de 0,0 est la moins dansable et 1,0 est la plus dansante.
- **Energie**: L'énergie est une mesure de 0,0 à 1,0 et représente une mesure perceptive de l'intensité et de l'activité. En règle générale, les pistes énergiques sont rapides et bruyantes. Par exemple, le death metal a une énergie élevée, tandis qu'un prélude de Bach obtient un score bas sur l'échelle. Les caractéristiques perceptives contribuant à cet attribut comprennent la plage dynamique, le volume sonore perçu, le timbre, la fréquence d'apparition et l'entropie générale.
- **Instrumentalness**: Prédit si une piste ne contient pas de voix. Les morceaux de rap ou de mots parlés sont clairement «vocaux». Plus la valeur de l'instrument est proche de 0, plus la piste est susceptible de ne contenir aucun contenu vocal. Les valeurs supérieures à 0,5 sont destinées à représenter des pistes instrumentales, mais la confiance est plus élevée lorsque la valeur s'approche de 1
- **Liveness**: détecte la présence d'un public dans l'enregistrement. Des valeurs de vivacité plus élevées représentent une probabilité accrue que la piste ait été jouée en direct. Une valeur supérieure à 0,8 offre une forte probabilité que la piste soit en direct.
- **Loudness**: le volume global d'une piste en décibels (dB). Les valeurs de sonie sont moyennées sur toute la piste et sont utiles pour comparer le volume relatif des pistes. Le volume est la qualité d'un son qui est le principal corrélat psychologique de la force physique (amplitude). Les valeurs sont généralement comprises entre -60 et 0 db.
- **Speechiness**: Speechiness détecte la présence de mots prononcés dans une piste. Plus l'enregistrement est exclusivement de type vocal (par exemple, talk-show, livre audio, poésie), plus la valeur d'attribut est proche de 1,0. Les valeurs supérieures à 0,66 décrivent des pistes qui sont probablement entièrement constituées de mots prononcés. Les valeurs comprises entre 0,33 et 0,66 décrivent des pistes qui peuvent contenir à la fois de la musique et de la parole, en sections ou en couches, y compris des cas tels que la musique rap. Les valeurs inférieures à 0,33 représentent très probablement de la musique et d'autres pistes non vocales.
- **Valence**: Une mesure de 0,0 à 1,0 décrivant la positivité musicale véhiculée par une piste. Les pistes à valence élevée semblent plus positives (par exemple, joyeuses, gaies, euphoriques), tandis que les pistes à faible valence semblent plus négatives (par exemple, tristes, déprimés, en colère).
- **Tempo**: Le tempo global estimé d'une piste en battements par minute (BPM). Dans la terminologie musicale, le tempo est la vitesse ou le rythme d'un morceau donné et dérive directement de la durée moyenne du battement.

