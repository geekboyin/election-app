<template>
  <div class="candidate-item">
    <span class="candidate-item__order">#{{ order + 1 }}</span>

    <div class="form-group" v-if="editing">
      <input type="text"
            :value="data.name"
            v-focus="editing"
            v-click-outside="outside"
            @keyup.enter="update($event)"
            @keyup.esc="editing = false"
      >
    </div>

    <span class="candidate-item__name"
          v-text="data.name"
          @click.prevent="editing = true"
          v-if="!editing"></span>

    <span class="candidate-item__actions">
      <button class="button button--primary button--vote-up" @click.prevent="voteUp(data.id)">+1</button>
      <button class="button button--vote-down"
              :disabled="data.votes === 0"
              @click.prevent="voteDown(data.id)"
      >-1</button>
    </span>
  </div>
</template>

<script>
import swal from 'sweetalert2'
export default {
  props: {
    data: {
      required: true
    },
    order: {
      type: Number,
      default: 0
    }
  },
  data () {
    return {
      editing: false
    }
  },
  methods: {
    update (event) {
      let input = event.target
      let name = input.value.trim()
      if (name === '') {
        // Delete candidate
        swal({
          title: '',
          text: this.t('candidate_delete_notice'),
          type: 'warning',
          showCancelButton: true,
          confirmButtonColor: '#3085d6',
          cancelButtonColor: '#d33',
          confirmButtonText: this.t('confirm_button_text'),
          cancelButtonText: this.t('cancel_button_text')
        }).then((result) => {
          if (result.value) {
            this.$store.dispatch('deleteCandidate', { id: this.data.id, name: name })
            swal(
              this.t('deleted_text'),
              '',
              'success'
            )
          }
        })
      } else {
        // Update candidate
        this.$store.dispatch('updateCandidate', { id: this.data.id, name: name })
      }
      this.editing = false
    },
    voteUp (id) {
      this.$store.dispatch('voteUp', id)
    },
    voteDown (id) {
      this.$store.dispatch('voteDown', id)
    },
    outside (e) {
      if (e.target.className === 'candidate-item__name') {
        return
      }
      this.editing = false
    }
  },
  directives: {
    focus (el, { value }, { context }) {
      if (value) {
        context.$nextTick(() => {
          el.focus()
        })
      }
    },
    'click-outside': {
      bind (el, binding, vNode) {
        if (typeof binding.value !== 'function') {
          console.warn(`${binding.express} is not a function`)
        }
        const bubble = binding.modifiers.bubble
        const handler = (e) => {
          if (bubble || (!el.contains(e.target) && el !== e.target)) {
            binding.value(e)
          }
        }
        el.__vueClickOutside__ = handler
        document.addEventListener('click', handler)
      },
      unbind (el, binding) {
        document.removeEventListener('click', el.__vueClickOutside__)
        el.__vueClickOutside__ = null
      }
    }
  }
}
</script>

<style lang="scss" scoped>
  .candidate-item {
    display: flex;
    flex-flow: row nowrap;
    align-items: center;
    padding: 10px 0;
    border-bottom: 1px solid $color--gray;
    &:last-child {
      border-bottom: none;
    }
    &__order {
      font-size: 1.8rem;
      font-weight: 700;
      padding-right: 10px;
      color: $color--gray;
    }
    &__name {
      font-size: 1.8rem;
      font-weight: 700;
      &:hover {
        cursor: pointer;
        background: $color--gray;
      }
    }
    &__actions {
      padding-left: 40px;
      margin-left: auto;
      .button:disabled {
        &:hover {
          background: transparent;
          color: initial;
        }
      }
      .button--vote-down {
        background-color: transparent;
        border: 1px solid $color--dark;
        color: $color--dark;
        &:hover {
          background-color: $color--dark;
          color: #fff;
        }
      }
    }
    .form-group {
      margin: 0;
    }
  }
</style>


