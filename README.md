 CADMEFE
 
 mkdir ibge-clone
cd ibge-clone
npm init -y
npm install express sequelize pg pg-hstore body-parser cors

npx create-react-app client
cd client
npm install axios react-leaflet leaflet

const express = require('express');
const bodyParser = require('body-parser');
const cors = require('cors');
const { Sequelize } = require('sequelize');
const app = express();
const port = 5000;

// Configurações
app.use(cors());
app.use(bodyParser.json());

// Conexão com o banco de dados
const sequelize = new Sequelize('postgres://user:password@localhost:5432/ibge_clone');

// Teste de conexão
sequelize.authenticate()
  .then(() => console.log('Conexão com o banco de dados estabelecida com sucesso.'))
  .catch(err => console.error('Não foi possível conectar ao banco de dados:', err));

// Definir rotas
app.get('/', (req, res) => {
  res.send('API funcionando!');
});

app.listen(port, () => {
  console.log(`Servidor rodando na porta ${port}`);
});

// models/Population.js
const { DataTypes } = require('sequelize');
const sequelize = require('../server'); // Certifique-se de exportar sua instância do Sequelize

const Population = sequelize.define('Population', {
  year: {
    type: DataTypes.INTEGER,
    allowNull: false
  },
  population: {
    type: DataTypes.INTEGER,
    allowNull: false
  }
});

module.exports = Population;

npx sequelize-cli migration:generate --name create-populations

'use strict';
module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.createTable('Populations', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER
      },
      year: {
        type: Sequelize.INTEGER,
        allowNull: false
      },
      population: {
        type: Sequelize.INTEGER,
        allowNull: false
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE
      }
    });
  },
  down: async (queryInterface, Sequelize) => {
    await queryInterface.dropTable('Populations');
  }
};

npx sequelize-cli db:migrate

// src/components/PopulationChart.js
import React, { useState, useEffect } from 'react';
import axios from 'axios';
import { Line } from 'react-chartjs-2';
import { Chart as ChartJS, LineElement, CategoryScale, LinearScale } from 'chart.js';

ChartJS.register(LineElement, CategoryScale, LinearScale);

const PopulationChart = () => {
  const [data, setData] = useState([]);

  useEffect(() => {
    axios.get('http://localhost:5000/populations')
      .then(response => {
        setData(response.data);
      })
      .catch(error => {
        console.error('Erro ao buscar dados:', error);
      });
  }, []);

  const chartData = {
    labels: data.map(item => item.year),
    datasets: [
      {
        label: 'População',
        data: data.map(item => item.population),
        borderColor: 'rgba(75,192,192,1)',
        backgroundColor: 'rgba(75,192,192,0.2)',
      }
    ]
  };

  return <Line data={chartData} />;
};

export default PopulationChart;

// src/components/Map.js
import React from 'react';
import { MapContainer, TileLayer, Marker, Popup } from 'react-leaflet';
import 'leaflet/dist/leaflet.css';

const Map = () => {
  return (
    <MapContainer center={[51.505, -0.09]} zoom={13} style={{ height: '500px', width: '100%' }}>
      <TileLayer
        url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
        attribution='&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
      />
      <Marker position={[51.505, -0.09]}>
        <Popup>
          A pretty CSS3 popup. <br /> Easily customizable.
        </Popup>
      </Marker>
    </MapContainer>
  );
};

export default Map;
