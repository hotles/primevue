<template>
    <transition name="p-contextmenusub" @enter="onEnter">
        <ul ref="container" :class="containerClass" role="menu" v-if="root ? true : parentActive">
            <template v-for="(item, i) of model">
                <li role="none" :class="getItemClass(item)" :style="item.style" v-if="item.visible !== false && !item.separator" :key="item.label + i"
                    @mouseenter="onItemMouseEnter($event, item)">
                    <router-link v-if="item.to" :to="item.to" :class="getLinkClass(item)" @click.native="onItemClick($event, item)" role="menuitem">
                        <span :class="['p-menuitem-icon', item.icon]"></span>
                        <span class="p-menuitem-text">{{item.label}}</span>
                    </router-link>
                    <a v-else :href="item.url" :class="getLinkClass(item)" :target="item.target" @click="onItemClick($event, item)"
                         :aria-haspopup="item.items != null" :aria-expanded="item === activeItem" role="menuitem"  :tabindex="item.disabled ? null : '0'">
                        <span :class="['p-menuitem-icon', item.icon]"></span>
                        <span class="p-menuitem-text">{{item.label}}</span>
                        <span class="p-submenu-icon pi pi-fw pi-caret-right" v-if="item.items"></span>
                    </a>
                    <sub-menu :model="item.items" v-if="item.visible !== false && item.items" :key="item.label + '_sub_'"
                        @leaf-click="onLeafClick" :parentActive="item === activeItem" />
                </li>
                <li class="p-menu-separator" :style="item.style" v-if="item.visible !== false && item.separator" :key="'separator' + i" role="separator"></li>
            </template>
        </ul>
    </transition>
</template>

<script>
import DomHandler from '../utils/DomHandler';

export default {
    name: 'sub-menu',
    props: {
        model: {
            type: Array,
            default: null
        },
        root: {
            type: Boolean,
            default: false
        },
        parentActive: {
            type: Boolean,
            default: false
        }
    },
    watch: {
        parentActive(newValue) {
            if (!newValue) {
                this.activeItem = null;
            }
        }
    },
    data() {
        return {
            activeItem: null
        }
    },
    methods: {
        onItemMouseEnter(event, item) {
            if (item.disabled) {
                event.preventDefault();
                return;
            }

            this.activeItem = item;
        },
        onItemClick(event, item) {
            if (item.disabled) {
                event.preventDefault();
                return;
            }

            if (!item.url && !item.to) {
                event.preventDefault();
            }

            if (item.command) {
                item.command({
                    originalEvent: event,
                    item: item
                });
            }

            if (!item.items) {
                this.onLeafClick();
            }
        },
        onLeafClick() {
            this.activeItem = null;
            this.$emit('leaf-click');
        },
        onEnter() {
            this.position();
        },
        position() {
            const parentItem = this.$refs.container.parentElement;
            const containerOffset = DomHandler.getOffset(this.$refs.container.parentElement)
            const viewport = DomHandler.getViewport();
            const sublistWidth = this.$refs.container.offsetParent ? this.$refs.container.offsetWidth : DomHandler.getHiddenElementOuterWidth(this.$refs.container);
            const itemOuterWidth = DomHandler.getOuterWidth(parentItem.children[0]);

            this.$refs.container.style.top = '0px';

            if ((parseInt(containerOffset.left, 10) + itemOuterWidth + sublistWidth) > (viewport.width - DomHandler.calculateScrollbarWidth())) {
                this.$refs.container.style.left = -1 * sublistWidth + 'px';
            }
            else {
                this.$refs.container.style.left = itemOuterWidth + 'px';
            }
        },
        getItemClass(item) {
            return [
                'p-menuitem', item.class, {
                    'p-menuitem-active': this.activeItem === item,
                    'p-disabled': item.disabled,
                }
            ]
        },
        getLinkClass(item) {
            return ['p-menuitem-link', {'p-disabled': item.disabled}];
        }
    },
    computed: {
        containerClass() {
            return {'p-submenu-list': !this.root};
        }
    }
}
</script>