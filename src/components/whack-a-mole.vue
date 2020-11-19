<template>
    <div class="whack-a-mole">
        <section class="game-area">
            <button
                class="hole"
                v-for="hole in holes"
                :key="hole"
                @click="whack(hole)"
            >
                <mole
                    :mole="moles.find(mole => mole.hole === hole) || {}"
                    @after-leave="afterLeave"
                    @killed="score++"
                />
            </button>
            
            <hammer />
        </section>


        <section class="info">
            <h1>🐹{{ score }}</h1>
            <button v-if="!playing" @click="initGame">Play ▶️</button>
            <button v-else @click="reset">Reset ⏹️</button>
        </section>
    </div>
</template>

<script>
import mole from "./mole";
import hammer from "./hammer";
export default {
    components: {
        mole,
        hammer,
    },
    data: () => ({
        holes: 9,
        maxMoles: 2,
        moles: [],
        animatingHoles: [],
        score: 0,
        playing: false,
        session: null,
        music: null,
    }),
    computed: {
        isMaxMoles() {
            return this.maxMoles <= this.holes && this.moles.length >= this.maxMoles
        }
    },
    methods: {
        reset() {
            clearInterval(this.session);
            this.score = 0;
            this.playing = false;
            this.music.pause();
            
        },
        afterLeave(hole) {
            const moleIndex = this.moles.findIndex(mole => mole.hole === hole);
            this.moles.splice(moleIndex, 1);
        },
        initGame() {
            this.music = new Audio(require('../assets/sanctuary_guardian.mp3'));
            this.music.loop = true;
            this.music.play();
            
            this.playing = true;
            
            this.session = setInterval(() => {
                this.spawnMole()
            }, 100);
        },
        whack(hole) {
            const moleIndex = this.moles.findIndex(mole => mole.hole === hole);
            const holeHasMole = moleIndex !== -1;

            if (holeHasMole) {
                console.log('whack!');
                clearTimeout(this.moles[moleIndex].timeout)
                this.moles[moleIndex].alive = false;
            }
        },
        spawnMole() {
            if (this.isMaxMoles) {
                return;
            }

            const busyHoles = new Set(this.moles.map(mole => mole.hole));
            let newMoleHole = this.randomHoleGenerator();

            while ( busyHoles.has(newMoleHole) ) {
               newMoleHole = this.randomHoleGenerator();
            }

            this.moles.push({
                hole: newMoleHole,
                alive: true,
                show: true,
                timeout: setTimeout(() => {
                    const moleIndex = this.moles.findIndex(mole => mole.hole === newMoleHole);
                    const mole = this.moles[moleIndex];
                    if (mole) {
                        clearTimeout(this.moles[moleIndex].timeout);
                        this.moles[moleIndex].show = false;
                    }
                }, 1000),
            })

        },
        randomHoleGenerator() {
            return Math.floor(Math.random() * this.holes) + 1;
        }
    },
}
</script>

<style lang="scss">
.whack-a-mole {
    height: 100vh;
    background-color: #272c42;
    .game-area {
        padding: 20px;
        cursor: none;
        display: grid;
        grid-template-columns: 1fr 1fr 1fr;
        grid-template-rows: 1fr 1fr 1fr;
        grid-gap: 70px 50px;
        .hole {
            height: 100px;
            border-radius: 50%;
            background: #7a7a7a;
            box-shadow: inset -1px 60px 18px 20px black;
            outline: none;
            overflow: hidden;
            border: 0;
            user-select: none;
            cursor: inherit;
        }
    }
    .info {
        display: grid;
        place-items: center;
        h1 {
            color: #fff;
            margin: 20px 0 0;
            font-size: 2.5em;
        }
        button {
            margin-top: 20px;
            border: 0;
            height: 30px;
            width: 100px;
            border-radius: 20px;
            background: #ffb900;
            font-size: 1em;
            font-weight: bold;
        }
    }
}
</style>