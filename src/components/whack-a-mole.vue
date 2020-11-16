<template>
  <div class="whack-a-mole">
        <button
            class="hole"
            v-for="hole in holes"
            :key="hole"
            @click="whack(hole)"
        >
            <transition
                name="mole-move"
                @after-leave="afterLeave(hole)"
            >
                <span v-if="moles.find(mole => mole.hole === hole && mole.alive)" class="mole">🐹</span>
            </transition>
        </button>
  </div>
</template>

<script>
export default {
    data: () => ({
        holes: 6,
        maxMoles: 2,
        moles: [],
        animatingHoles: [],
    }),
    created() {
        this.initGame();
    },
    computed: {
        isMaxMoles() {
            return this.maxMoles <= this.holes && this.moles.length >= this.maxMoles
        }
    },
    methods: {
        afterLeave(hole) {
            const moleIndex = this.moles.findIndex(mole => mole.hole === hole);
            this.moles.splice(moleIndex, 1);
        },
        initGame() {
            setInterval(() => {
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
                timeout: setTimeout(() => {
                    const moleIndex = this.moles.findIndex(mole => mole.hole === newMoleHole);
                    const mole = this.moles[moleIndex];
                    if (mole) {
                        clearTimeout(this.moles[moleIndex].timeout);
                        this.moles[moleIndex].alive = false;
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
        font-size: 100px;
        overflow: hidden;
    }

    .mole {
        display: block;
        line-height: 110px;
    }

    .mole-move-enter-active {
        transition: transform 600ms cubic-bezier(0.65, 0.05, 0.36, 1)
    }
    .mole-move-leave-active {
        transition: transform 150ms ease-out;
    }
    .mole-move-enter {
        transform: translateY(100px);
    }
    .mole-move-leave-to {
        transform: translateY(100px);
    }
}
</style>