<template>
    <div class="user" @click="openMenu">
        <img :src="admin ? '/admin.jpg' : '/user.jpg'" />
        <span :class="{ local: local }">{{ name }}</span>
        <teleport to="#app">
            <context-menu
                v-if="menuShow"
                ref="ctxMenu"
                class="ctx-menu"
                :left="menuLeft"
                :top="menuTop"
                :items="options"
                @action="action"
            ></context-menu>
            <popup-dialog
                v-if="rename"
                @close="() => (rename = false)"
                :title="`Change nickname for ${name}`"
            >
                <input
                    type="text"
                    v-model="newName"
                    :class="{
                        input: true,
                        invalid: !(nameValid.len && nameValid.chars),
                    }"
                    placeholder="New nickname"
                />
                <div class="rules">
                    <span>New nickname:</span>
                    <ul>
                        <li :class="{ invalid: !nameValid.len }">
                            Must be 3-24 chars long
                        </li>
                        <li :class="{ invalid: !nameValid.chars }">
                            Can contain only A-Z, 0-9, - and _
                        </li>
                    </ul>
                </div>
                <div :class="{button:true, disabled: !(nameValid.len && nameValid.chars)}" @click="renamee">Apply</div>
            </popup-dialog>
        </teleport>
    </div>
</template>

<script>
/* eslint-disable */
//keep eslint from being annoying (indent issues on switch statements)
import constants from '../constants.js';

import ContextMenu from "./ContextMenu.vue";
import PopupDialog from "./PopupDialog.vue";
export default {
    components: {
        ContextMenu,
        PopupDialog,
    },
    $refs: {
        ctxMenu: HTMLDivElement,
    },
    data() {
        return {
            menuShow: false,
            menuLeft: 0,
            menuTop: 0,
            options: [
                { name: "kick", text: "Kick", icon: "close", enabled: false },
                {
                    name: "promote",
                    text: "Promote",
                    icon: "admin",
                    enabled: false,
                },
                {
                    name: "rename",
                    text: "Change nickname",
                    icon: "edit",
                    enabled: false,
                },
            ],
            nameValid: {
                len: false,
                chars: false,
            },
            rename: false,
            newName: "",
        };
    },
    watch: {
        newName(newVal, oldVal) {
            this.nameValid.len = constants.nameRegexes.len.test(newVal);
            this.nameValid.chars = constants.nameRegexes.chars.test(newVal);

        },
    },
    mounted() {
        this.options[0].enabled = this.islocaladmin && !this.local;
        this.options[1].enabled = this.islocaladmin && !this.local;
        this.options[2].enabled = this.local;
    },
    methods: {
        openMenu(e) {
            console.log(e.which);
            switch (e.which) {
                case 2:
                    return;
                case 3:
                    e.preventDefault();
                    break;
            }
            if (this.menuShow) return;

            this.options[0].enabled = this.islocaladmin && !this.local;
            this.options[1].enabled = this.islocaladmin && !this.local;
            this.options[2].enabled = this.local;

            this.menuShow = true;
            this.$nextTick(() => {
                this.menuLeft = e.clientX;
                this.menuTop = e.clientY;
                requestAnimationFrame(() => {
                    document.addEventListener("click", this.closeMenu);
                });
            });
        },
        closeMenu(e) {
            document.removeEventListener("click", this.closeMenu);
            this.menuShow = false;
        },
        action(a) {
            switch (a) {
                case "kick":
                    this.$emit("kick", this.id);
                    break;
                case "promote":
                    this.$emit("promote", this.id);
                    break;
                case "rename":
                    this.rename = true;
                    break;
            }
        },
        renamee() {
            if (!(this.nameValid.len && this.nameValid.chars)) return;
            
            this.rename = false;
            this.$emit("rename", this.newName);
        },
    },
    props: {
        name: { type: String, required: true },
        id: { type: String, required: true },
        admin: { type: Boolean, required: true },
        local: { type: Boolean, required: true },
        islocaladmin: { type: Boolean, required: true },
    },
};
</script>

<style lang="less">
@import url("@/assets/theme.less");

.user {
    user-select: none;

    display: flex;
    flex-direction: column;

    position: relative;

    align-items: center;
    align-content: center;
    justify-items: center;
    justify-content: center;
    border-radius: 10px;

    img {
        width: 64px;
        height: 64px;
        border-radius: 50%;
    }
    span {
        width: 96px;
        font-size: 12px;
        text-align: center;
    }
}

.user:hover {
    transition: 100ms ease;
    background-color: @background-light;
}

.local {
    color: @accent !important;
}

.ctx-menu {
    position: absolute;
    z-index: 5;
    width: fit-content;
    height: fit-content;
    left: 50%;
    right: 50%;
}

.rules {
    padding: 0;
    margin: 10px 0;

    ul {
        margin: 0;
        li {
            color: @background-light;
            &.invalid {
                color: @error;
            }
        }
    }
}
</style>
