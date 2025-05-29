# Usage Examples

## Photo Upload

### Basic Upload Component
```tsx
import { useArtWall } from "@/contexts/artwall-context"
import { DragDropZone } from "@/components/drag-drop-zone"

export function UploadPage() {
  const { uploadPhotos, loading, error } = useArtWall()

  return (
    <div className="p-4">
      <h1 className="text-2xl font-bold mb-4">Upload Photos</h1>
      
      <DragDropZone
        onDrop={uploadPhotos}
        loading={loading}
        error={error}
      />
      
      {error && (
        <div className="text-red-500 mt-2">
          {error}
        </div>
      )}
    </div>
  )
}
```

### Photo Gallery
```tsx
import { useArtWall } from "@/contexts/artwall-context"

export function PhotoGallery() {
  const { photos, removePhoto } = useArtWall()

  return (
    <div className="grid grid-cols-3 gap-4 p-4">
      {photos.map(photo => (
        <div key={photo.id} className="relative group">
          <img
            src={photo.url}
            alt={photo.name}
            className="w-full h-48 object-cover rounded"
          />
          
          <button
            onClick={() => removePhoto(photo.id)}
            className="absolute top-2 right-2 opacity-0 group-hover:opacity-100
                     bg-red-500 text-white p-2 rounded-full"
          >
            Remove
          </button>
        </div>
      ))}
    </div>
  )
}
```

## Layout Planning

### Layout Selector
```tsx
import { useArtWall } from "@/contexts/artwall-context"
import type { LayoutType } from "@/hooks/useLayout"

const layouts: LayoutType[] = ["grid", "salon", "row", "symmetric"]

export function LayoutSelector() {
  const { layoutType, setLayoutType, photos, generateLayout } = useArtWall()

  const handleLayoutChange = (type: LayoutType) => {
    setLayoutType(type)
    generateLayout(photos, type)
  }

  return (
    <div className="flex gap-4 p-4">
      {layouts.map(type => (
        <button
          key={type}
          onClick={() => handleLayoutChange(type)}
          className={`px-4 py-2 rounded ${
            layoutType === type
              ? "bg-black text-white"
              : "bg-gray-200 hover:bg-gray-300"
          }`}
        >
          {type.charAt(0).toUpperCase() + type.slice(1)}
        </button>
      ))}
    </div>
  )
}
```

### Wall Canvas
```tsx
import { useArtWall } from "@/contexts/artwall-context"
import { WallCanvas } from "@/components/wall-canvas"
import { FrameControls } from "@/components/frame-controls"

export function PlannerPage() {
  const {
    frames,
    updateFrame,
    removeFrame,
    wallDimensions,
    setWallDimensions
  } = useArtWall()

  return (
    <div className="flex h-screen">
      <div className="flex-1 relative">
        <WallCanvas
          frames={frames}
          dimensions={wallDimensions}
          onFrameUpdate={updateFrame}
          onFrameRemove={removeFrame}
        />
      </div>
      
      <div className="w-80 border-l p-4">
        <FrameControls
          dimensions={wallDimensions}
          onDimensionsChange={setWallDimensions}
        />
      </div>
    </div>
  )
}
```

## AR Preview

### AR View Component
```tsx
import { useArtWall } from "@/contexts/artwall-context"
import { useEffect, useRef } from "react"

export function ARPreview() {
  const { frames } = useArtWall()
  const videoRef = useRef<HTMLVideoElement>(null)
  
  useEffect(() => {
    if (!videoRef.current) return
    
    // Request camera access
    navigator.mediaDevices
      .getUserMedia({ video: { facingMode: "environment" } })
      .then(stream => {
        if (videoRef.current) {
          videoRef.current.srcObject = stream
        }
      })
      .catch(console.error)
      
    return () => {
      const stream = videoRef.current?.srcObject as MediaStream
      stream?.getTracks().forEach(track => track.stop())
    }
  }, [])

  return (
    <div className="relative h-screen">
      <video
        ref={videoRef}
        autoPlay
        playsInline
        className="w-full h-full object-cover"
      />
      
      <div className="absolute inset-0">
        {frames.map(frame => (
          <div
            key={frame.id}
            style={{
              position: "absolute",
              top: frame.y,
              left: frame.x,
              width: frame.width,
              height: frame.height,
              transform: `rotate(${frame.rotation}deg)`,
              border: "2px solid white",
              boxShadow: "0 0 10px rgba(0,0,0,0.5)"
            }}
          />
        ))}
      </div>
    </div>
  )
}
```

## Advanced Features

### Frame Manipulation
```tsx
import { useArtWall } from "@/contexts/artwall-context"
import type { Frame } from "@/hooks/useLayout"

export function FrameEditor({ frame }: { frame: Frame }) {
  const { updateFrame } = useArtWall()

  const handleRotation = (angle: number) => {
    updateFrame({
      ...frame,
      rotation: frame.rotation + angle
    })
  }

  const handleResize = (scale: number) => {
    updateFrame({
      ...frame,
      width: frame.width * scale,
      height: frame.height * scale
    })
  }

  return (
    <div className="absolute top-2 right-2 flex gap-2">
      <button
        onClick={() => handleRotation(-5)}
        className="p-2 bg-white rounded-full shadow"
      >
        ↶
      </button>
      
      <button
        onClick={() => handleRotation(5)}
        className="p-2 bg-white rounded-full shadow"
      >
        ↷
      </button>
      
      <button
        onClick={() => handleResize(0.9)}
        className="p-2 bg-white rounded-full shadow"
      >
        -
      </button>
      
      <button
        onClick={() => handleResize(1.1)}
        className="p-2 bg-white rounded-full shadow"
      >
        +
      </button>
    </div>
  )
}
```

### Export Functionality
```tsx
import { useArtWall } from "@/contexts/artwall-context"
import { exportToImage } from "@/lib/export-utils"

export function ExportButton() {
  const { frames, photos, wallDimensions } = useArtWall()

  const handleExport = async () => {
    try {
      const canvas = document.createElement("canvas")
      canvas.width = wallDimensions.width
      canvas.height = wallDimensions.height
      
      const ctx = canvas.getContext("2d")
      if (!ctx) return
      
      // Draw background
      ctx.fillStyle = "white"
      ctx.fillRect(0, 0, canvas.width, canvas.height)
      
      // Draw frames
      for (const frame of frames) {
        const photo = photos.find(p => p.id === frame.photoId)
        if (!photo) continue
        
        ctx.save()
        ctx.translate(frame.x + frame.width/2, frame.y + frame.height/2)
        ctx.rotate(frame.rotation * Math.PI / 180)
        ctx.drawImage(
          await loadImage(photo.url),
          -frame.width/2,
          -frame.height/2,
          frame.width,
          frame.height
        )
        ctx.restore()
      }
      
      // Download image
      const link = document.createElement("a")
      link.download = "artwall.png"
      link.href = canvas.toDataURL()
      link.click()
    } catch (error) {
      console.error("Export failed:", error)
    }
  }

  return (
    <button
      onClick={handleExport}
      className="px-4 py-2 bg-black text-white rounded"
    >
      Export Layout
    </button>
  )
}

function loadImage(url: string): Promise<HTMLImageElement> {
  return new Promise((resolve, reject) => {
    const img = new Image()
    img.onload = () => resolve(img)
    img.onerror = reject
    img.src = url
  })
}
