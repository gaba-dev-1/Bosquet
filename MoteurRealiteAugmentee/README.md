<div align="center">

# 👓 Moteur de Réalité Augmentée Économique

**Visualisation AR pour économie jardinière en temps réel**

[![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)]()
[![WebAR](https://img.shields.io/badge/WebAR-FF6B6B?style=for-the-badge&logo=augmented-reality&logoColor=white)]()
[![Three.js](https://img.shields.io/badge/Three.js-000000?style=for-the-badge&logo=three.js&logoColor=white)]()

*"Voir l'économie réelle à travers les yeux de la nature"*

</div>

### Implémentation du Moteur AR

```typescript
import React, { useState, useEffect, useRef } from 'react';
import * as THREE from 'three';
import { Canvas, useFrame, useThree } from '@react-three/fiber';
import { Html, Text, Sphere, Box } from '@react-three/drei';

interface EcologicalValue {
  carbonSequestered: number; // kg CO₂
  biodiversityIndex: number; // nombre d'espèces
  soilHealth: number; // 0-100
  waterPurified: number; // litres
  economicValue: number; // euros
  cosmicAlignment: {
    itaca: number;
    bifrost: number;
    ouroboros: number;
  };
}

interface GardenLocation {
  id: string;
  coordinates: [number, number, number];
  ecologicalValue: EcologicalValue;
  gardenerName: string;
  lastUpdate: Date;
  growthTrend: 'increasing' | 'stable' | 'decreasing';
}

const ARGardenEconomyViewer: React.FC = () => {
  const [gardenData, setGardenData] = useState<GardenLocation[]>([]);
  const [selectedGarden, setSelectedGarden] = useState<GardenLocation | null>(null);
  const [realTimeValues, setRealTimeValues] = useState<Map<string, EcologicalValue>>(new Map());

  // Simulation de données de jardins en temps réel
  useEffect(() => {
    const mockGardens: GardenLocation[] = [
      {
        id: 'lyon-parc-tete-or',
        coordinates: [0, 0, 0],
        ecologicalValue: {
          carbonSequestered: 156.7,
          biodiversityIndex: 89,
          soilHealth: 87,
          waterPurified: 2340,
          economicValue: 2847.50,
          cosmicAlignment: { itaca: 8.7, bifrost: 9.2, ouroboros: 7.9 }
        },
        gardenerName: 'Marie Dubois',
        lastUpdate: new Date(),
        growthTrend: 'increasing'
      },
      {
        id: 'lyon-croix-rousse',
        coordinates: [5, 2, -3],
        ecologicalValue: {
          carbonSequestered: 234.1,
          biodiversityIndex: 134,
          soilHealth: 92,
          waterPurified: 3450,
          economicValue: 4234.80,
          cosmicAlignment: { itaca: 9.1, bifrost: 8.8, ouroboros: 9.5 }
        },
        gardenerName: 'Paul Martin',
        lastUpdate: new Date(),
        growthTrend: 'increasing'
      },
      {
        id: 'lyon-presquile',
        coordinates: [-3, 1, 4],
        ecologicalValue: {
          carbonSequestered: 98.3,
          biodiversityIndex: 67,
          soilHealth: 79,
          waterPurified: 1890,
          economicValue: 1967.30,
          cosmicAlignment: { itaca: 7.8, bifrost: 8.3, ouroboros: 8.1 }
        },
        gardenerName: 'Sophie Laurent',
        lastUpdate: new Date(),
        growthTrend: 'stable'
      }
    ];

    setGardenData(mockGardens);

    // Simulation de mise à jour en temps réel
    const updateInterval = setInterval(() => {
      setGardenData(prevData => 
        prevData.map(garden => ({
          ...garden,
          ecologicalValue: {
            ...garden.ecologicalValue,
            carbonSequestered: garden.ecologicalValue.carbonSequestered + Math.random() * 0.5,
            economicValue: garden.ecologicalValue.economicValue + Math.random() * 10,
            biodiversityIndex: Math.max(0, garden.ecologicalValue.biodiversityIndex + Math.random() * 2 - 1)
          },
          lastUpdate: new Date()
        }))
      );
    }, 5000);

    return () => clearInterval(updateInterval);
  }, []);

  return (
    <div className="ar-garden-economy-viewer">
      <Canvas camera={{ position: [0, 10, 15], fov: 75 }}>
        <ambientLight intensity={0.6} />
        <pointLight position={[10, 10, 10]} intensity={1} />
        
        {/* Grille de base représentant l'espace urbain */}
        <gridHelper args={[20, 20, '#4CAF50', '#81C784']} />
        
        {/* Visualisation des jardins */}
        {gardenData.map(garden => (
          <GardenVisualization
            key={garden.id}
            garden={garden}
            isSelected={selectedGarden?.id === garden.id}
            onSelect={setSelectedGarden}
          />
        ))}
        
        {/* Interface de contrôle AR */}
        <ARControlInterface 
          selectedGarden={selectedGarden}
          totalEconomicValue={gardenData.reduce((sum, g) => sum + g.ecologicalValue.economicValue, 0)}
        />
      </Canvas>
      
      {/* Interface de données en temps réel */}
      <EconomicDataOverlay gardens={gardenData} selectedGarden={selectedGarden} />
    </div>
  );
};

const GardenVisualization: React.FC<{
  garden: GardenLocation;
  isSelected: boolean;
  onSelect: (garden: GardenLocation) => void;
}> = ({ garden, isSelected, onSelect }) => {
  const meshRef = useRef<THREE.Mesh>(null);
  const [hovered, setHovered] = useState(false);

  useFrame((state) => {
    if (meshRef.current) {
      // Animation de respiration pour montrer la vie du jardin
      const breathe = Math.sin(state.clock.elapsedTime * 2) * 0.1 + 1;
      meshRef.current.scale.setScalar(breathe);
      
      // Rotation douce pour l'esthétique
      meshRef.current.rotation.y += 0.005;
    }
  });

  // Couleur basée sur la santé écologique
  const getGardenColor = () => {
    const health = garden.ecologicalValue.soilHealth / 100;
    return new THREE.Color().setHSL(health * 0.3, 0.8, 0.5); // Du rouge au vert
  };

  // Taille basée sur la valeur économique
  const getGardenSize = () => {
    return Math.log(garden.ecologicalValue.economicValue) * 0.3;
  };

  return (
    <group position={garden.coordinates}>
      {/* Sphère principale représentant le jardin */}
      <Sphere
        ref={meshRef}
        args={[getGardenSize(), 16, 16]}
        position={[0, 0, 0]}
        onClick={() => onSelect(garden)}
        onPointerOver={() => setHovered(true)}
        onPointerOut={() => setHovered(false)}
      >
        <meshPhongMaterial 
          color={getGardenColor()} 
          transparent 
          opacity={isSelected ? 0.9 : 0.7}
          emissive={hovered ? '#222' : '#000'}
        />
      </Sphere>
      
      {/* Particules représentant la biodiversité */}
      <BiodiversityParticles count={garden.ecologicalValue.biodiversityIndex} />
      
      {/* Flux de carbone visualisé */}
      <CarbonFlowVisualization carbonRate={garden.ecologicalValue.carbonSequestered} />
      
      {/* Étiquette avec informations économiques */}
      <Html position={[0, getGardenSize() + 2, 0]} center>
        <div className="garden-label">
          <div className="gardener-name">{garden.gardenerName}</div>
          <div className="economic-value">💰 {garden.ecologicalValue.economicValue.toFixed(2)}€</div>
          <div className="carbon-info">🌱 {garden.ecologicalValue.carbonSequestered.toFixed(1)} kg CO₂</div>
          <div className="biodiversity-info">🦋 {garden.ecologicalValue.biodiversityIndex} espèces</div>
        </div>
      </Html>
      
      {/* Connexions cosmiques (lignes vers les autres jardins) */}
      {isSelected && <CosmicConnections currentGarden={garden} />}
    </group>
  );
};

const BiodiversityParticles: React.FC<{ count: number }> = ({ count }) => {
  const particlesRef = useRef<THREE.Points>(null);
  
  useEffect(() => {
    if (particlesRef.current) {
      const geometry = new THREE.BufferGeometry();
      const positions = new Float32Array(count * 3);
      const colors = new Float32Array(count * 3);
      
      for (let i = 0; i < count; i++) {
        // Position aléatoire autour du jardin
        positions[i * 3] = (Math.random() - 0.5) * 4;
        positions[i * 3 + 1] = Math.random() * 3;
        positions[i * 3 + 2] = (Math.random() - 0.5) * 4;
        
        // Couleurs variées pour représenter différentes espèces
        colors[i * 3] = Math.random();
        colors[i * 3 + 1] = Math.random();
        colors[i * 3 + 2] = Math.random();
      }
      
      geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
      geometry.setAttribute('color', new THREE.BufferAttribute(colors, 3));
    }
  }, [count]);

  useFrame(() => {
    if (particlesRef.current) {
      particlesRef.current.rotation.y += 0.002;
    }
  });

  return (
    <points ref={particlesRef}>
      <bufferGeometry />
      <pointsMaterial size={0.05} vertexColors transparent opacity={0.8} />
    </points>
  );
};

const CarbonFlowVisualization: React.FC<{ carbonRate: number }> = ({ carbonRate }) => {
  const flowRef = useRef<THREE.Group>(null);
  
  useFrame((state) => {
    if (flowRef.current) {
      // Flux de carbone visualisé comme des spirales vertes montantes
      flowRef.current.children.forEach((child, index) => {
        child.position.y = (state.clock.elapsedTime * 2 + index) % 6;
        child.rotation.y += 0.1;
      });
    }
  });

  // Créer des éléments de flux basés sur le taux de carbone
  const flowElements = Array.from({ length: Math.min(Math.floor(carbonRate / 50), 10) }, (_, i) => (
    <Box key={i} args={[0.1, 0.1, 0.1]} position={[Math.sin(i) * 2, 0, Math.cos(i) * 2]}>
      <meshBasicMaterial color="#4CAF50" transparent opacity={0.6} />
    </Box>
  ));

  return <group ref={flowRef}>{flowElements}</group>;
};

const CosmicConnections: React.FC<{ currentGarden: GardenLocation }> = ({ currentGarden }) => {
  // Visualisation des connexions avec la trinité cosmique
  const { itaca, bifrost, ouroboros } = currentGarden.ecologicalValue.cosmicAlignment;
  
  return (
    <group>
      {/* Ligne vers Itaca (cohésion) */}
      <Html position={[0, 4, 0]} center>
        <div className="cosmic-connection">
          <div style={{ color: '#2196F3' }}>🏝️ Itaca Cohésion: {itaca.toFixed(1)}/10</div>
        </div>
      </Html>
      
      {/* Ligne vers Bifrost (multiniveau) */}
      <Html position={[0, 3, 0]} center>
        <div className="cosmic-connection">
          <div style={{ color: '#9C27B0' }}>🌈 Bifrost Multi: {bifrost.toFixed(1)}/10</div>
        </div>
      </Html>
      
      {/* Ligne vers Ouroboros (transformation) */}
      <Html position={[0, 2, 0]} center>
        <div className="cosmic-connection">
          <div style={{ color: '#FF5722' }}>🐉 Ouroboros Trans: {ouroboros.toFixed(1)}/10</div>
        </div>
      </Html>
    </group>
  );
};

const ARControlInterface: React.FC<{
  selectedGarden: GardenLocation | null;
  totalEconomicValue: number;
}> = ({ selectedGarden, totalEconomicValue }) => {
  return (
    <Html position={[0, 8, 0]} center>
      <div className="ar-control-interface">
        <h2>🌱 Économie Jardinière Lyon AR</h2>
        <div className="total-value">
          Valeur Économique Totale: <strong>{totalEconomicValue.toFixed(2)}€</strong>
        </div>
        {selectedGarden && (
          <div className="selected-garden-details">
            <h3>{selectedGarden.gardenerName}</h3>
            <p>Tendance: {selectedGarden.growthTrend === 'increasing' ? '📈' : 
                        selectedGarden.growthTrend === 'stable' ? '➡️' : '📉'}</p>
            <p>Dernière mise à jour: {selectedGarden.lastUpdate.toLocaleTimeString()}</p>
          </div>
        )}
      </div>
    </Html>
  );
};

const EconomicDataOverlay: React.FC<{
  gardens: GardenLocation[];
  selectedGarden: GardenLocation | null;
}> = ({ gardens, selectedGarden }) => {
  const totalCarbon = gardens.reduce((sum, g) => sum + g.ecologicalValue.carbonSequestered, 0);
  const totalBiodiversity = gardens.reduce((sum, g) => sum + g.ecologicalValue.biodiversityIndex, 0);
  const totalEconomic = gardens.reduce((sum, g) => sum + g.ecologicalValue.economicValue, 0);

  return (
    <div className="economic-data-overlay">
      <div className="stats-panel">
        <h3>📊 Statistiques Économie Jardinière</h3>
        <div className="stat-item">
          <span className="stat-label">💰 Valeur Totale:</span>
          <span className="stat-value">{totalEconomic.toFixed(2)}€</span>
        </div>
        <div className="stat-item">
          <span className="stat-label">🌱 CO₂ Séquestré:</span>
          <span className="stat-value">{totalCarbon.toFixed(1)} kg</span>
        </div>
        <div className="stat-item">
          <span className="stat-label">🦋 Biodiversité:</span>
          <span className="stat-value">{totalBiodiversity} espèces</span>
        </div>
        <div className="stat-item">
          <span className="stat-label">👥 Jardiniers Actifs:</span>
          <span className="stat-value">{gardens.length}</span>
        </div>
      </div>
      
      {selectedGarden && (
        <div className="selected-garden-panel">
          <h3>🎯 Jardin Sélectionné</h3>
          <p><strong>Jardinier:</strong> {selectedGarden.gardenerName}</p>
          <p><strong>Revenus Aujourd'hui:</strong> {selectedGarden.ecologicalValue.economicValue.toFixed(2)}€</p>
          <p><strong>Santé du Sol:</strong> {selectedGarden.ecologicalValue.soilHealth}%</p>
          <p><strong>Eau Purifiée:</strong> {selectedGarden.ecologicalValue.waterPurified}L</p>
          
          <h4>🌟 Alignement Cosmique</h4>
          <div className="cosmic-alignment">
            <div>🏝️ Itaca: {selectedGarden.ecologicalValue.cosmicAlignment.itaca.toFixed(1)}/10</div>
            <div>🌈 Bifrost: {selectedGarden.ecologicalValue.cosmicAlignment.bifrost.toFixed(1)}/10</div>
            <div>🐉 Ouroboros: {selectedGarden.ecologicalValue.cosmicAlignment.ouroboros.toFixed(1)}/10</div>
          </div>
        </div>
      )}
    </div>
  );
};

export default ARGardenEconomyViewer;
```

