<template>
    <Lunchbox
        :background="'white'"
        :cameraPosition="[2, 2.6, 5]"
        :cameraLook="[0, 0, 0]"
        orthographic
        shadow
    >
        <!-- Lights -->
        <directionalLight castShadow :position="[0, 10, 10]" />
        <ambientLight :intensity="0.2" />

        <!-- Ground -->
        <mesh
            name="ground"
            :rotation-x="Math.PI * -0.5"
            :scale="10"
            receiveShadow
        >
            <planeGeometry />
            <meshStandardMaterial color="brown" />
        </mesh>

        <!-- Player -->
        <group
            :position-x="playerPos.x"
            :position-y="0.5"
            :position-z="playerPos.z"
            :scale="0.2"
        >
            <mesh castShadow>
                <sphereGeometry />
                <meshStandardMaterial
                    :color="playerCharge > 0 ? 'yellow' : 'green'"
                />
            </mesh>
        </group>

        <!-- Enemies -->
        <group v-for="enemy in enemies" :key="enemy.uuid">
            <mesh
                castShadow
                :scale="0.3"
                :position="[
                    enemy.position.x,
                    enemy.position.y,
                    enemy.position.z,
                ]"
            >
                <boxGeometry />
                <meshStandardMaterial
                    :color="playerCharge > 0 ? 'tomato' : 'white'"
                />
            </mesh>
        </group>

        <!-- Zones -->
        <group
            v-for="zone in zones"
            :key="zone.uuid"
            :rotation-y="Math.sin(now * 0.003)"
            :position="[zone.position.x, zone.position.y, zone.position.z]"
        >
            <mesh
                castShadow
                :scale-x="0.1"
                :scale-y="0.2"
                :scale-z="zone.scale.z"
                :position-x="-zone.scale.x * 0.5 + 0.05"
            >
                <boxGeometry />
                <meshStandardMaterial color="yellow" />
            </mesh>
            <mesh
                castShadow
                :scale-x="0.1"
                :scale-y="0.2"
                :scale-z="zone.scale.z"
                :position-x="zone.scale.x * 0.5 - 0.05"
            >
                <boxGeometry />
                <meshStandardMaterial color="yellow" />
            </mesh>
            <mesh
                castShadow
                :scale-x="zone.scale.x"
                :scale-y="0.2"
                :scale-z="0.1"
                :position-z="-zone.scale.z * 0.5"
            >
                <boxGeometry />
                <meshStandardMaterial color="yellow" />
            </mesh>
            <mesh
                castShadow
                :scale-x="zone.scale.x"
                :scale-y="0.2"
                :scale-z="0.1"
                :position-z="zone.scale.z * 0.5"
            >
                <boxGeometry />
                <meshStandardMaterial color="yellow" />
            </mesh>
        </group>
    </Lunchbox>
</template>

<script setup lang="ts">
import { computed, onMounted, ref, watch } from 'vue'
import { camera, onBeforeRender } from 'lunchboxjs'
import * as THREE from 'three'

const size = 10
const now = ref(0)
let last = Date.now()

// Enemies
let latest = 0
const enemies = ref<
    {
        uuid: number
        position: { x: number; y: number; z: number }
        velocity: number
        kind: 'enemy'
    }[]
>([])
const addEnemy = () => {
    enemies.value.push({
        uuid: latest++,
        position: {
            x: size * -0.5 + Math.random() * size * -0.5,
            y: 0.5,
            z: Math.random() * size - size * 0.5,
        },
        velocity: Math.random() * 0.01 + 0.01,
        kind: 'enemy',
    })
}

// Zones
const zones = ref<
    {
        uuid: number
        scale: { x: number; z: number }
        position: { x: number; y: number; z: number }
        kind: 'zone'
    }[]
>([])
const addZone = () => {
    zones.value.push({
        uuid: latest++,
        position: {
            x: Math.random() * size * 0.9 - size * 0.45,
            y: 0.1,
            z: Math.random() * size * 0.9 - size * 0.45,
        },
        scale: { x: 0.85, z: 0.85 },
        kind: 'zone',
    })
}
addZone()

// Charge
const playerCharge = ref(0)
watch(playerCharge, (v) => {
    if (v === 0) {
        addZone()
    }
})

onMounted(() => {
    ;(camera.value as THREE.OrthographicCamera)!.zoom = 1.5
})

const playerPos = ref({ x: 0, z: 0 })
const velocity = ref({ x: 0, z: 0 })

const timeScale = ref(1)

const dir = ref({ up: 0, right: 0, down: 0, left: 0 })
const accel = { x: 0.002, z: 0.002 }
const max = { x: 0.05, z: 0.05 }
const scaleChange = ref(0)
window.addEventListener('keydown', (evt) => {
    switch (evt.key) {
        case 'ArrowUp':
        case 'w':
            dir.value.up = 1
            // dir.value.left = 1
            break
        case 'ArrowRight':
        case 'd':
            dir.value.right = 1
            // dir.value.up = 1
            break
        case 'ArrowDown':
        case 's':
            dir.value.down = 1
            // dir.value.right = 1
            break
        case 'ArrowLeft':
        case 'a':
            dir.value.left = 1
            // dir.value.down = 1
            break

        case ' ':
            timeScale.value = 0.1
            break

        case 'q':
            scaleChange.value = 1
            break
        case 'e':
            scaleChange.value = -1
            break
    }
})
window.addEventListener('keyup', (evt) => {
    switch (evt.key) {
        case 'ArrowUp':
        case 'w':
            dir.value.up = 0
            // dir.value.left = 0
            break
        case 'ArrowRight':
        case 'd':
            dir.value.right = 0
            // dir.value.up = 0
            break
        case 'ArrowDown':
        case 's':
            dir.value.down = 0
            // dir.value.right = 0
            break
        case 'ArrowLeft':
        case 'a':
            dir.value.left = 0
            // dir.value.down = 0
            break

        case ' ':
            timeScale.value = 1
            break

        case 'q':
            scaleChange.value = 0
            break
        case 'e':
            scaleChange.value = 0
            break
    }
})

const empty1 = new THREE.Vector2()
const empty2 = new THREE.Vector2()

onBeforeRender(() => {
    playerPos.value.x += velocity.value.x * timeScale.value
    playerPos.value.z += velocity.value.z * timeScale.value
    velocity.value.z -= accel.z * dir.value.up
    velocity.value.z += accel.z * dir.value.down
    velocity.value.x += accel.x * dir.value.right
    velocity.value.x -= accel.x * dir.value.left

    velocity.value.x = Math.max(Math.min(velocity.value.x, max.z), -max.x)
    velocity.value.z = Math.max(Math.min(velocity.value.z, max.z), -max.z)

    velocity.value.x *= 0.95
    velocity.value.z *= 0.95

    // enemies move across screen
    enemies.value.forEach((enemy) => {
        enemy.position.x += enemy.velocity * timeScale.value
    })
    enemies.value = enemies.value.filter((e) => e.position.x <= size * 0.6)
    // make sure we have at least a few enemies
    while (enemies.value.length < 4) {
        addEnemy()
    }

    // update time
    const delta = (Date.now() - last) * timeScale.value
    now.value += delta
    last = Date.now()

    // check collisions
    empty1.x = playerPos.value.x
    empty1.y = playerPos.value.z
    const collisions = [...zones.value, ...enemies.value].filter((item) => {
        empty2.x = item.position.x
        empty2.y = item.position.z
        return empty1.distanceTo(empty2) <= 0.5
    })
    collisions.forEach((coll) => {
        if (coll.kind === 'zone') {
            zones.value.splice(
                zones.value.findIndex((z) => z.uuid === coll.uuid),
                1
            )
            playerCharge.value++
        } else if (coll.kind === 'enemy') {
            if (playerCharge.value > 0) {
                playerCharge.value--
                enemies.value.splice(
                    enemies.value.findIndex((z) => z.uuid === coll.uuid),
                    1
                )
            }
        }
    })
})
</script>