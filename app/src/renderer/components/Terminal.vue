<template>
    <div class="Main">

        <div class="Titlebar">{{isFullScreen?lastCommand:pwd}}</div>

        <div class="Console" v-show="!isFullScreen" @click="clickConsole">
            <div class="Console-inner" ref="inner">
                <div v-for="op of log">
                    <div class="Console-input">
                        <Formula v-model="op.inputFormula" />
                    </div>
                    <div class="Console-output">{{op.outputString}}</div>
                </div>
                <div class="Input Console-input" v-show="!isRunning">
                    <Formula v-model="inputForm" :writable="true" @send="send" ref="inputForm" @y="y" 
                        @onfocus="onfocus"
                        @onblur="onblur"
                        :inputText.sync="inputText"
                    />
                </div>
            </div>
        </div>

        <div ref="xterm" class="xterm" :style="{'z-index':isFullScreen?'1000':'-10','opacity':isFullScreen?'1':'0'}"></div>

        <Recomend :input="inputForm" :inputText.sync="inputText" @update="mlupdate" />
    </div>
</template>

<style>
.Main {
    position: absolute;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    background: rgba(0, 0, 0, 0.6);
}

.xterm {
    position: absolute;
    top: 36px;
}

.Titlebar {
    position: fixed;
    height: 36px;
    top: 0px;
    left: 0;
    right: 0;
    z-index: 200;
    -webkit-app-region: drag;
    text-align: center;
    line-height: 36px;
    color: rgba(255, 255, 255, 0.7);
}

.Console {
    position: absolute;
    top: 0px;
    bottom: 0px;
    left: 0px;
    right: 0px;
    /* border-radius: 3px; */
    color: #fff;
    cursor: text;
    user-select: text;
}
.Console-inner {
    position: absolute;
    bottom: 15px;
    left: 15px;
    right: 15px;
    /* padding-right: 5px; */
    top: 36px;
    overflow: scroll;
}

.Console-input {
    margin-bottom: 5px;
    line-height: 24px;
}
.Input {
    margin-bottom: 0;
}

.Console-output {
    margin-bottom: 5px;
    white-space: pre-wrap;
}
</style>

<script>
import Recomend from "@/components/Recomend"
import Formula from "@/components/Formula"

const { ipcRenderer } = require("electron")
const child_process = require("child_process")
const path = require("path")
import { Terminal } from "xterm"
import * as fit from "xterm/lib/addons/fit/fit"
Terminal.applyAddon(fit)

const HOMEDIR =
    process.env[process.platform == "win32" ? "USERPROFILE" : "HOME"]

let xterm
let child


export default {
    components: {
        Recomend,
        Formula
    },
    mounted() {
        this.focus()

        xterm = new Terminal({
            allowTransparency: true,
            theme: {
                background: "transparent"
            }
        })

        xterm.open(this.$refs.xterm)

        xterm.on("data", data => {
            child.stdin.write(data)
        })

        this.fits()
        window.addEventListener("resize", this.fits)
    },
    data() {
        return {
            candidate: [],
            log: [],
            inputForm: [
                // {
                //     type: "text",
                //     val: "a"
                // }
            ],
            inputArr: [],
            pwd: HOMEDIR,

            isFullScreen: false,
            lastCommand: "",
            isRunning: false,

            inputText: "",

            suggestY: 0
        }
    },
    watch: {
        inputForm: {
            deep: true,
            handler() {
                this.inputArr = this.formulaToArray(this.inputForm)
                // console.log("水素の音:", this.inputForm)
                // console.log(this.inputArr)
            }
        },
        log: {
            deep: true,
            handler() {
                this.scrollToBottom()
            }
        },
        value() {
            this.scrollToBottom()
        }
    },
    created() {
        // 受信処理
        ipcRenderer.on("inputBlock", (event, block) => {
            this.inputBlock(block)
        })
    },
    methods: {
        updateInputText(text) {
            this.inputText = text
        },
        clickConsole() {
            this.focus()
        },
        focus() {
            if (this.$refs.inputForm) this.$refs.inputForm.focus()
        },
        blur() {
            if (this.$refs.inputForm) this.$refs.inputForm.focus()
        },
        scrollToBottom() {
            this.$nextTick(() => {
                this.$refs.inner.scrollTop = this.$refs.inner.scrollHeight
            })
        },
        onfocus() {
            ipcRenderer.send("onfocus")
        },
        onblur() {
            ipcRenderer.send("onblur")
        },
        y(r) {
            this.suggestY = r
            if (r.top > 1) {
                ipcRenderer.send("setSubWindowBounds", {
                    x: r.left,
                    y: r.top
                })
            }
        },
        inputBlock(block) {
            this.inputForm.push(JSON.parse(JSON.stringify(block)))
            this.$refs.inputForm.focus()
        },
        formulaToArray(input) {
            let s = []

            for (let block of input) {
                s.push(block.val)
            }

            return s
        },
        send() {
            const f = JSON.parse(JSON.stringify(this.inputForm))

            const ioobj = {
                inputFormula: f,
                outputString: ""
            }

            this.log.push(ioobj)
            this.inputForm.splice(0)

            this.scrollToBottom()

            const arr = this.inputArr

            // console.log(arr)

            //const ls = child_process.spawn(a[0], a.slice(1))
            // console.log(this.inputArr)
            child = child_process.exec(arr.join(" "), {
                cwd: this.pwd,
                shell: true
                // cols: xterm.cols,
                // rows: xterm.rows
            })

            this.isRunning = true

            xterm.focus()

            this.lastCommand = arr[0]

            if (arr[0].match(/vim?|less|sl|top/)) {
                xterm.clear()
                this.isFullScreen = true
            }

            child.stdout.on("data", data => {
                if (!this.isFullScreen) ioobj.outputString += data

                xterm.write(data)
            })
            child.stderr.on("data", data => {
                if (!this.isFullScreen) ioobj.outputString += data
            })

            child.on("close", code => {
                this.isRunning = false
                this.isFullScreen = false

                if (code === 0) {
                    if (arr[0] == "cd") {
                        const p = arr[1]

                        let n = ""

                        if (p[0] == "~") {
                            n = HOMEDIR + "/" + p.slice(1)
                        } else if (p[0] == "/") {
                            n = p
                        } else {
                            n = this.pwd + "/" + p
                        }

                        this.pwd = path.normalize(n)
                    } else if (arr[0] == "clear" || arr[0] == "cls") {
                        this.log = []
                    }
                }
                // console.log(`child process exited with code ${code}`)

                this.focus()
            })

            
        },
        mlupdate(list) {
            this.candidate = list

            // 非同期
            ipcRenderer.send("candidateList", list)
        },
        fits() {
            // xterm.fit()
            // if (child) child.resize(xterm.cols, xterm.rows)
        }
    }
}
</script>
