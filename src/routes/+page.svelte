<script lang="ts">
    let imageInput: HTMLInputElement
    let canvas: HTMLCanvasElement
    let ctx: CanvasRenderingContext2D
    let originalImage = $state<HTMLImageElement | null>(null)
    let ditheredImage = $state<string | null>(null)
    let hasImage = $derived(!!originalImage)

    $effect(() => {
        if (canvas) {
            ctx = canvas.getContext('2d')!
        }
    })

    const handleImageUpload = (e: Event) => {
        const file = (e.target as HTMLInputElement).files?.[0]
        if (file) {
            const reader = new FileReader()
            reader.onload = (e: ProgressEvent<FileReader>) => {
                const img = new Image()
                img.onload = () => {
                    originalImage = img
                    canvas.width = img.width
                    canvas.height = img.height
                    ctx.drawImage(img, 0, 0)
                    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height)
                    const ditheredData = ditherImage(imageData)
                    ctx.putImageData(ditheredData, 0, 0)
                    ditheredImage = canvas.toDataURL()
                }
                img.src = e.target?.result as string
            }
            reader.readAsDataURL(file)
        }
    }

    const ditherImage = (imageData: ImageData): ImageData => {
        const data = imageData.data
        const width = imageData.width
        const height = imageData.height

        for (let y = 0; y < height; y++) {
            for (let x = 0; x < width; x++) {
                const i = (y * width + x) * 4
                const avg = (data[i] + data[i + 1] + data[i + 2]) / 3
                const newColor = avg > 128 ? 255 : 0
                const err = avg - newColor
                data[i] = data[i + 1] = data[i + 2] = newColor
                if (x + 1 < width) {
                    data[i + 4] += (err * 7) / 16
                    data[i + 5] += (err * 7) / 16
                    data[i + 6] += (err * 7) / 16
                }
                if (y + 1 < height) {
                    if (x > 0) {
                        data[i + width * 4 - 4] += (err * 3) / 16
                        data[i + width * 4 - 3] += (err * 3) / 16
                        data[i + width * 4 - 2] += (err * 3) / 16
                    }
                    data[i + width * 4] += (err * 5) / 16
                    data[i + width * 4 + 1] += (err * 5) / 16
                    data[i + width * 4 + 2] += (err * 5) / 16
                    if (x + 1 < width) {
                        data[i + width * 4 + 4] += (err * 1) / 16
                        data[i + width * 4 + 5] += (err * 1) / 16
                        data[i + width * 4 + 6] += (err * 1) / 16
                    }
                }
            }
        }
        return imageData
    }
</script>

<h1>Dither Rams yourself</h1>
<input type="file" accept="image/*" onchange={handleImageUpload} bind:this={imageInput} />
<canvas bind:this={canvas} style="display: none;"></canvas>
{#if ditheredImage}
    <h2>Dithered Image:</h2>
    <img src={ditheredImage} alt="Dithered" />
{/if}
