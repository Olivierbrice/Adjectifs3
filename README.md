<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Les suites de nombres</title>
    <style>
        body {
            font-family: 'Comic Sans MS', cursive, sans-serif;
            text-align: center;
            background-color: #f0f8ff;
            padding: 20px;
        }
        h1 {
            color: #ff6b6b;
            font-size: 2.5em;
            margin-bottom: 30px;
        }
        .number-container {
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            gap: 15px;
            margin: 30px auto;
            max-width: 800px;
        }
        .number {
            background-color: #74b9ff;
            color: white;
            font-size: 2em;
            width: 80px;
            height: 80px;
            display: flex;
            align-items: center;
            justify-content: center;
            border-radius: 10px;
            cursor: pointer;
            transition: all 0.3s;
            box-shadow: 0 4px 8px rgba(0,0,0,0.1);
        }
        .number:hover {
            transform: scale(1.1);
            background-color: #0984e3;
        }
        .feedback {
            font-size: 1.5em;
            margin: 20px;
            min-height: 60px;
        }
        .correct {
            color: #00b894;
        }
        .incorrect {
            color: #d63031;
        }
        .next-btn {
            background-color: #00b894;
            color: white;
            border: none;
            padding: 10px 20px;
            font-size: 1.2em;
            border-radius: 5px;
            cursor: pointer;
            margin-top: 20px;
            display: none;
        }
        .next-btn:hover {
            background-color: #009975;
        }
        .progress {
            margin-top: 20px;
            font-size: 1.2em;
            color: #636e72;
        }
        .final-score {
            font-size: 2em;
            color: #6c5ce7;
            margin: 30px 0;
        }
        .restart-btn {
            background-color: #6c5ce7;
            color: white;
            border: none;
            padding: 10px 20px;
            font-size: 1.2em;
            border-radius: 5px;
            cursor: pointer;
            margin-top: 20px;
        }
        .restart-btn:hover {
            background-color: #5649be;
        }
    </style>
</head>
<body>
    <div id="game-container">
        <h1>Les suites de nombres</h1>
        <div class="progress" id="progress">Question 1/6</div>
        <div class="feedback" id="feedback"></div>
        <div class="number-container" id="number-container"></div>
        <button class="next-btn" id="next-btn">Suivant</button>
    </div>

    <script>
        // Variables du jeu
        let currentPage = 0;
        let score = 0;
        const totalPages = 6;
        let currentSequence = [];
        let errorPosition = -1;
        let stepSize = 0;

        // Éléments du DOM
        const numberContainer = document.getElementById('number-container');
        const feedbackEl = document.getElementById('feedback');
        const nextBtn = document.getElementById('next-btn');
        const progressEl = document.getElementById('progress');
        const gameContainer = document.getElementById('game-container');

        // Fonction pour générer une séquence avec une erreur
        function generateSequence() {
            // Déterminer le pas (+2 pour la première page, puis aléatoire parmi +3, +4, +5, +10)
            if (currentPage === 0) {
                stepSize = 2;
            } else {
                const possibleSteps = [3, 4, 5, 10];
                stepSize = possibleSteps[Math.floor(Math.random() * possibleSteps.length)];
            }

            // Générer un nombre de départ entre 20 et (200 - stepSize*9)
            const startNumber = Math.floor(Math.random() * (180 - stepSize * 9)) + 20;
            
            // Créer la séquence correcte
            const correctSequence = [];
            for (let i = 0; i < 10; i++) {
                correctSequence.push(startNumber + i * stepSize);
            }

            // Choisir une position aléatoire pour l'erreur (sauf la première)
            errorPosition = Math.floor(Math.random() * 9) + 1;
            
            // Créer une séquence avec erreur
            currentSequence = [...correctSequence];
            
            // Modifier le nombre à la position d'erreur
            // On s'assure que l'erreur n'est pas trop évidente (pas le même nombre que le précédent)
            let errorValue;
            do {
                errorValue = currentSequence[errorPosition - 1] + Math.floor(Math.random() * stepSize * 2) + 1;
            } while (errorValue === currentSequence[errorPosition - 1] || 
                     errorValue === correctSequence[errorPosition] ||
                     Math.abs(errorValue - correctSequence[errorPosition]) < 2);

            currentSequence[errorPosition] = errorValue;

            return {
                sequence: currentSequence,
                correctNumber: correctSequence[errorPosition],
                step: stepSize
            };
        }

        // Fonction pour afficher la séquence
        function displaySequence(sequence) {
            numberContainer.innerHTML = '';
            sequence.forEach((num, index) => {
                const numEl = document.createElement('div');
                numEl.className = 'number';
                numEl.textContent = num;
                numEl.dataset.index = index;
                numEl.addEventListener('click', handleNumberClick);
                numberContainer.appendChild(numEl);
            });
        }

        // Gestion du clic sur un nombre
        function handleNumberClick(e) {
            const clickedIndex = parseInt(e.target.dataset.index);
            
            // Désactiver tous les nombres
            document.querySelectorAll('.number').forEach(num => {
                num.style.pointerEvents = 'none';
            });

            if (clickedIndex === errorPosition) {
                // Bonne réponse
                e.target.style.backgroundColor = '#00b894';
                feedbackEl.textContent = `Bravo ! La bonne réponse était ${currentSequence[errorPosition]}.`;
                feedbackEl.className = 'feedback correct';
                score++;
            } else {
                // Mauvaise réponse
                e.target.style.backgroundColor = '#d63031';
                document.querySelector(`.number[data-index="${errorPosition}"]`).style.backgroundColor = '#00b894';
                feedbackEl.textContent = `Presque ! Le nombre incorrect était ${currentSequence[errorPosition]}. La bonne valeur aurait dû être ${currentSequence[errorPosition - 1] + stepSize}.`;
                feedbackEl.className = 'feedback incorrect';
            }

            // Afficher le bouton suivant
            nextBtn.style.display = 'inline-block';
        }

        // Fonction pour passer à la page suivante
        function nextPage() {
            currentPage++;
            progressEl.textContent = `Question ${currentPage + 1}/${totalPages}`;
            
            if (currentPage < totalPages) {
                // Réinitialiser pour la nouvelle question
                feedbackEl.textContent = '';
                feedbackEl.className = 'feedback';
                nextBtn.style.display = 'none';
                
                // Générer et afficher une nouvelle séquence
                const { sequence } = generateSequence();
                displaySequence(sequence);
            } else {
                // Afficher le score final
                showFinalScore();
            }
        }

        // Fonction pour afficher le score final
        function showFinalScore() {
            gameContainer.innerHTML = `
                <h1>Les suites de nombres</h1>
                <div class="final-score">Ton score: ${score}/${totalPages}</div>
                <div class="feedback">${getFeedbackMessage(score)}</div>
                <button class="restart-btn" id="restart-btn">Recommencer</button>
            `;
            
            document.getElementById('restart-btn').addEventListener('click', restartGame);
        }

        // Fonction pour obtenir un message de feedback basé sur le score
        function getFeedbackMessage(score) {
            const percentage = (score / totalPages) * 100;
            
            if (percentage >= 90) {
                return "Excellent travail ! Tu es un champion des suites de nombres !";
            } else if (percentage >= 70) {
                return "Très bien ! Tu as bien compris le principe des suites.";
            } else if (percentage >= 50) {
                return "Pas mal ! Avec un peu plus de pratique, tu vas devenir excellent.";
            } else {
                return "Continue à t'entraîner ! Tu vas y arriver.";
            }
        }

        // Fonction pour recommencer le jeu
        function restartGame() {
            currentPage = 0;
            score = 0;
            progressEl.textContent = `Question 1/${totalPages}`;
            feedbackEl.textContent = '';
            feedbackEl.className = 'feedback';
            
            // Générer et afficher une nouvelle séquence
            const { sequence } = generateSequence();
            displaySequence(sequence);
            
            // Réafficher le contenu du jeu
            gameContainer.innerHTML = `
                <h1>Les suites de nombres</h1>
                <div class="progress" id="progress">Question 1/${totalPages}</div>
                <div class="feedback" id="feedback"></div>
                <div class="number-container" id="number-container"></div>
                <button class="next-btn" id="next-btn">Suivant</button>
            `;
            
            // Réattacher les événements
            numberContainer = document.getElementById('number-container');
            feedbackEl = document.getElementById('feedback');
            nextBtn = document.getElementById('next-btn');
            progressEl = document.getElementById('progress');
            
            displaySequence(sequence);
            nextBtn.addEventListener('click', nextPage);
        }

        // Initialisation du jeu
        window.onload = function() {
            const { sequence } = generateSequence();
            displaySequence(sequence);
            nextBtn.addEventListener('click', nextPage);
        };
    </script>
</body>
</html>
