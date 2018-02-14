<template>
    <main class="main">
        <div class="world">
            <canvas id="canvas"></canvas>
        </div>
        <div id="log" class="log">
            <div class="logline" v-for="message in messages">
                {{ message }}
            </div>
        </div>

    </main>
</template>

<script>
import paper from 'paper';

export default {
    name: 'GameComponent',
    data () {
        return {
            world: null,
            messages: []
        }
    },
    created() {
        
    },
    mounted() {
        paper.install(window);
        paper.setup(document.getElementById('canvas'));

        this.connect();
    },
    methods: {
        connect() {

            this.log('Connecting... ');

            this.ws = new WebSocket('ws://localhost:8001/ws');
            this.ws.onopen = () => {
                this.log('Connected');
                this.ws.addEventListener('message', (e) => {
                    var msg = JSON.parse(e.data);
                    this.log('< '+(msg.message || msg.data));
                    this.executeMessage(msg);
                });

                this.send({type:'getworld'});
            }
            this.ws.onclose = () => {
                this.log('Connection closed');
            }
            this.ws.onerror = () => {
                this.log('Connection error');
            }
        },

        send(msg) {
            this.log('> ' + msg.type);
            this.ws.send(
                JSON.stringify(msg)
            )
        },

        executeMessage(msg) {

            switch(msg.type) {

                case 'world':
                    var path = new paper.Path();
                    path.strokeColor = 'black';
                    var start = new paper.Point(20, 20);
                    path.moveTo(start);
                    path.lineTo(start.add([ 20, 10 ]));
                    path.lineTo(start.add([ 0, 20 ]));
                    path.lineTo(start.add([ 0, 0 ]));
                    paper.view.draw();
                    break;

            }

        },

        log(msg) {
            console.log(msg);
            this.messages.push(msg);
        }
    }

}
</script>

<style scoped>
    .main {
        display: flex;
        flex-direction: row;
        justify-content: flex-start;
        align-items: stretch;
        align-content: stretch;

        width: 100%;
        height: 100%;
    }
    .world {
        flex: 1;

        border: 1px solid black;
        padding: 10px;
        margin: 5px;
    }
    .log {
        flex: 1;

        display: flex;
        flex-direction: column;
        justify-content: flex-start;

        border: 1px solid black;
        padding: 10px;
        margin: 5px;
        overflow-y: scroll;
    }
    .logline {
        color: gray;
    }

    #canvas {
        width: 500px;
        height: 500px;
        background-color: rgb(236, 236, 236);
    }
</style>
