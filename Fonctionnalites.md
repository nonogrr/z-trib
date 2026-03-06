# Z-Trib 

## Présentation
Application pour la gestion d'événements sportif qui 
  - met en relation des sportifs amateurs pour organiser des matchs, des tournois, des entrainements
  - permet pour un club d'organiser des évenements

## Présentation

**Compte d'un membre**
  - Nom / Pseudo
  - Age
  - Region
  - Adresse mail
  - Sports pratiqués
  - Formations / diplomes sportif
  - Un avatar
  - Des stats sur les événements auxquels il aparticipé
  - Son classement général avec des points appelés "tribs"
  - Visbilité : profil public, profil privé, profil visible par les membres de mes tribus
  - Le classement selon les tribs (global ou par sport)
  - mon stats par activités sportives et par participation à des événements
  - Authentification / Inscription : gmail, apple, mail/password
     - MFA en option : appli / mail
  
**Les tribus**
  - Groupe de sportif réunis autours d'un ou plusieurs sports
  - Les tribus sont composées de membre avec des chefs de tribu qui gere la tribu (invitation, revocation, modification des informations)

**Les évenements sont de sur 3 niveaux de confidentialités**
  - Un évenement "privé"
    - Uniquement un membre peut créer un évenement "privé"
    - Le membre qui créé l'evement est gestionnaire de l'evenement
      - il peut déléguer à d'autres membres
    - Les gestionnaires de l'evenement ont les droits suivants
      - Droits de configuration de l'evenement avec la date, la périodicité, le lieu, le sport, l'activité (tournoi, entrainement), le visuel (logo, bandeau), le reglement, les modalités d'accès et l'organisation (vestiaire, restauration, ....)
      - Droits de suppression, cloture de l'evenement
    - L'évenement est visible unqiuement aux membres
    - Tous les membres de la tribu peuvent s'inscrire
    - Tous les membres de la tribu sont autonommes pour rentrer les scores si on est sur un tounroi
    
  - Un évenement "tribu"
    - Uniquement un membre peut créer un évenement qui sera associée à une tribu
    - Le membre qui créé l'evement est gestionnaire de l'evenement
      - il peut déléguer à d'autres membres de la tribu
    - Les gestionnaires de l'evenement ont les droits suivants
      - Droits de configuration de l'evenement avec la date, la périodicité, le lieu, le sport, l'activité (tournoi, entrainement), le visuel (logo, bandeau), le reglement, les modalités d'accès et l'organisation (vestiaire, restauration, ....)
      - Droits de suppression, cloture de l'evenement
    - L'évenement est visible unqiuement aux membres de l'évenement.
    - pas d'inscription possible
    - Tous les membres de la tribu sont autonommes pour rentrer les scores si on est sur un tounroi
    
  - Un événement "public/autonomme"
    - Un évement ouvert au public
    - Uniquement un membre peut créer un évenement qui sera associée à une tribu
    - Le membre qui créé l'evement est gestionnaire de l'evenement
      - Il peut déléguer à d'autres membres (membre de n'importe quel tribus à laquelle il appartient)
    - Les gestionnaires de l'evenement ont les droits suivants
      - Droits de configuration de l'evenement avec la date, la périodicité, le lieu, le sport, l'activité (tournoi, entrainement), le visuel (logo, bandeau), le reglement, les modalités d'accès et l'organisation (vestiaire, restauration, ....)
      - Inscription des équipes / joueurs qui ne sont pas des membres ou fournir un lien d'inscription pour que les équipes joueurs renseigne une fiche informative
      - Rentrer les résultats
      - Communiquer des informations durant le trounoi
    - Un evenement public permet d'avoir plusieurs page publique via lien partagé avec qrcode (Full page sans les menus en mode déconnecté via lien partagé)
      - Page pour les inscriptions
      - Page avec les informations utiles pour le jour de l'évenement (modalité d'accés, repartition dans les vestiaires, reglement, restauration sur place)
      - Page avec les matchs terminés, en cours et à venir, le classement avec un filtre par joueur/equipe pour pouvoir voir uniquement ce qui concerne le joueur ou l'équipe
      - Page informative sur l existe une page publique avec un lien partagé(génération d'un QR code) - Tu peux filtrer sur cette page par équipe engagée dans l'evenement pour qu'il voit sa poule, ses resultats, ses matchs à venir - En haut, tu retrouves le nom de l'evenement avec le logo, le status (en cours, terminé) et le QR CODE - tu as également toutes les informations comme le reglement, l'organisation, les choses disponibles sur place, infos de derniere minute
  
**Les activités**
  - Une compétition avec des matchs, des scores et un classement avec les possibilités suivantes :
    -  Un championnat (tout le monde rencontre tout le monde)
    -  Un match unique
    -  Un tournoi avec phase de poule suivi d'une phase éliminatoire
    -  Un tournoi avec phase de poule suivi d'une seconde pahse de poule pour déterminer le classement
    -  Un tournoi avec montante / descendante
  - Un entrainement / activité ludique sans scores et classement
    -  Juste une description de ce qui va être fait.

**Les sports**
  - le nom du sport
  - Le logo
  - uniquement les globals admin peuvent rentrer un sport
  - les modalités d'une rencontre
    - Nombre de participant à une rencontre (2 ou plus)  
    - Type de rencontre
      - match avec score
      - match avec des sets et/ou des jeux
      - match sans score comme pour les echecs uniquement Victoire, Nul, Défaire
      - Classement à l'arrivée avec ou sans chrono
     
