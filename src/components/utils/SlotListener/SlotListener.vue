<script lang="ts">
import { defineComponent } from 'vue'
import type { VNode, PropType } from 'vue'
import type { SlotListenerTrigger, TriggerEventHandlers } from '@/components/utils/SlotListener/types'
import { getFirstSlotVNode } from '@/utils/getFirstSlotNode'
import { pick } from 'lodash-es'

// inspired from https://github.com/TuSimple/naive-ui/blob/main/src/popover/src/Popover.tsx

const triggerEventMap: Record<SlotListenerTrigger, string[]> = {
  focus: ['onFocus', 'onBlur'],
  click: ['onClick'],
  hover: ['onMouseenter', 'onMouseleave'],
}

function appendEvents(
    vNode: VNode,
    events: TriggerEventHandlers,
): void {
  Object.entries(triggerEventMap).forEach(([, eventNames]) => {
    eventNames.forEach((eventName) => {
      if (!vNode.props) vNode.props = {}
      else {
        vNode.props = Object.assign({}, vNode.props)
      }
      const originalHandler = vNode.props[eventName]
      const handler = events[eventName as keyof typeof events]
      if (!originalHandler) vNode.props[eventName] = handler
      else {
        vNode.props[eventName] = (...args: unknown[]) => {
          originalHandler(...args)
          // eslint-disable-next-line @typescript-eslint/no-explicit-any
          ;(handler as any)(...args)
        }
      }
    })
  })
}

export default defineComponent({
  name: 'SlotListener',
  emits: ['click', 'focus', 'blur', 'mouseenter', 'mouseleave'],
  props: {
    trigger: {
      type: String as PropType<SlotListenerTrigger>,
      default: 'click',
    },
  },
  setup(props, { emit }) {
    const handleFocus = (e: FocusEvent) => {
      emit('focus', e)
    }
    const handleBlur = (e: FocusEvent) => {
      emit('blur', e)
    }
    const handleClick = (e: MouseEvent) => {
      emit('click', e)
    }
    const handleMouseEnter = (e: MouseEvent) => {
      emit('mouseenter', e)
    }
    const handleMouseLeave = (e: MouseEvent) => {
      emit('mouseleave', e)
    }

    return {
      handleClick,
      handleBlur,
      handleFocus,
      handleMouseLeave,
      handleMouseEnter,
    }
  },
  render() {
    const {
      $slots,
    } = this

    const handlers = {
      onClick: this.handleClick,
      onMouseenter: this.handleMouseEnter,
      onMouseleave: this.handleMouseLeave,
      onFocus: this.handleFocus,
      onBlur: this.handleBlur,
    }

    let triggerVNode = getFirstSlotVNode($slots, 'default')

    const ascendantAndCurrentHandlers: TriggerEventHandlers[] = [
      handlers,
    ]
    if (triggerVNode?.props)
      ascendantAndCurrentHandlers.push(pick(triggerVNode.props, 'onClick', 'onMouseenter', 'onMouseleave', 'onFocus', 'onBlur'))

    const mergedHandlers: TriggerEventHandlers = {
      onBlur: (e: FocusEvent) => {
        ascendantAndCurrentHandlers.forEach((_handlers) => {
          _handlers?.onBlur?.(e)
        })
      },
      onFocus: (e: FocusEvent) => {
        ascendantAndCurrentHandlers.forEach((_handlers) => {
          _handlers?.onFocus?.(e)
        })
      },
      onClick: (e: MouseEvent) => {
        ascendantAndCurrentHandlers.forEach((_handlers) => {
          _handlers?.onClick?.(e)
        })
      },
      onMouseenter: (e: MouseEvent) => {
        ascendantAndCurrentHandlers.forEach((_handlers) => {
          _handlers?.onMouseenter?.(e)
        })
      },
      onMouseleave: (e: MouseEvent) => {
        ascendantAndCurrentHandlers.forEach((_handlers) => {
          _handlers?.onMouseleave?.(e)
        })
      },
    }

    if (triggerVNode)
      appendEvents(
          triggerVNode,
          mergedHandlers,
      )

    return triggerVNode
  },
})
</script>
