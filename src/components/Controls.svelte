<script>
    import { createEventDispatcher } from "svelte";
    const dispatch = createEventDispatcher();

    const handleKeydown = (event) => {
        let code = event.keyCode;
        let direction = "";

        if (code === 82) dispatch("restart");
        if (code === 69) dispatch("toggleEffects");
        if (code === 84) dispatch("speedChange");

        switch (code) {
            case 87:
            case 38:
                direction = "up";
                break;
            case 68:
            case 39:
                direction = "right";
                break;
            case 83:
            case 40:
                direction = "down";
                break;
            case 65:
            case 37:
                direction = "left";
                break;
            default:
                break;
        }

        if (direction) dispatch("directionChange", direction);
    };
</script>

<svelte:window on:keydown|preventDefault={handleKeydown} />
