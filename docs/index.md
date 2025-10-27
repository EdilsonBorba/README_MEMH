
# Análises biomecânicas em vídeo (CMJ, Gait Stance, Pronação)

Scripts **auditáveis** com OpenCV/MediaPipe/SciPy para extrair medidas **angulares** e **espaço‑temporais** em tarefas motoras:

- **CMJ / Vertical Jump** (sagital, 1 perna)  
- **Gait Stance** (sagital, 1 perna — TD→TO)  
- **Pronação manual** (vista posterior, marcação 4 pontos)

Saídas padronizadas: **vídeo com overlay**, **gráficos (PNG)** e **uma planilha Excel consolidada** em `Dados_Gerais/` (com abas específicas por análise).

> **Orientação de vídeo**: os analisadores **não forçam rotação**; aplicam apenas a **orientação do metadata** do arquivo (quando presente).  
> **Padrão de pastas (minimalista)**: `Videos/`, `Graficos/`, `Dados_Gerais/`.

---

## Requisitos (Python 3.9+)

```
pip install opencv-python mediapipe numpy pandas matplotlib scipy xlsxwriter
```

---

## Fluxo geral de uso

1) Escolha o **vídeo** e a **pasta de saída**.  
2) Informe o **nome do sujeito/teste** (usado nos arquivos).  
3) Se solicitado, informe parâmetros (ex.: **perna analisada**; **massa corporal** e **coxa (cm)** no Gait Stance).  
4) O processamento salva arquivos nas pastas **Videos/**, **Graficos/** e **Dados_Gerais/**.

---

# 1) Gait Stance (sagital, 1 perna)

Detecta **TD → TO** de **uma perna** em vista sagital, extrai ângulos de **joelho/tornozelo**, e (opcionalmente) calcula **velocidade do quadril** (px/s e m/s) e **energia cinética** (J) usando **massa corporal** e **escala** por coxa.

### Passo a passo
1. Selecionar vídeo e pasta de saída.  
2. Digitar o nome do sujeito/teste.  
3. Escolher **LEFT/RIGHT**.  
4. (Opcional) Informar **massa (kg)** e **comprimento real da coxa (cm)** para escalar velocidades (m/s) e energia (J).  
5. O script faz TD/TO; se falhar, gera um XLSX de **debug** (série completa + flag de contato).

### Saídas
- `Videos/NOME_overlay_SIDE.mp4` — esqueleto + ângulos; faixa verde no topo durante **apoio** (TD→TO).  
- `Graficos/NOME_norm_plot_SIDE.png` — **joelho** e **tornozelo** (filtrados) normalizados **0–100%** do apoio.  
- `Dados_Gerais/NOME_STANCE_SIDE_dados_gerais.xlsx` (um único arquivo com abas):  
  - **Discretos** (apoio identificado)  
  - **Continuo** (séries do apoio)  
  - **Norm100** (0–100% do apoio)  
  - **Bruto_Marcado** (XY dos pontos durante o apoio)

### Dicionário de dados — Gait Stance

#### Aba **Discretos**
| Coluna | Significado |
|---|---|
| `name`, `side` | Identificação do teste e perna (`LEFT`/`RIGHT`). |
| `TD_frame`, `TO_frame` | Frames de toque e saída. |
| `TD_time_s`, `TO_time_s` | Tempos absolutos de TD e TO (s). |
| `support_time_s` | Duração do apoio (s). |
| `knee_min_deg`, `knee_max_deg`, `knee_ROM_deg` | Mín., máx. e amplitude do joelho (°). |
| `knee_at_TD_deg`, `knee_at_TO_deg` | Ângulos do joelho nos eventos (°). |
| `ankle_min_deg`, `ankle_max_deg`, `ankle_ROM_deg` | Mín., máx. e amplitude do tornozelo (°). |
| `ankle_at_TD_deg`, `ankle_at_TO_deg` | Ângulos do tornozelo nos eventos (°). |
| `hip_speed_mean_px_s`, `hip_speed_peak_px_s` | Velocidade média e pico do quadril (px/s). |
| `hip_speed_mean_m_s`, `hip_speed_peak_m_s` | Velocidade média e pico do quadril (m/s) — requer escala. |
| `hip_KE_mean_J`, `hip_KE_peak_J` | Energia cinética média e pico (J) — requer massa + escala. |
| `body_mass_kg` | Massa corporal usada (kg). |
| `thigh_cm_input` | Comprimento real da coxa informado (cm). |
| `thigh_px_median_stance` | Mediana Hip→Knee (px) no apoio. |
| `cm_per_px_est` | Fator de escala `cm/px`. |

#### Aba **Continuo**
| Coluna | Significado |
|---|---|
| `t_rel_s`, `t_abs_s` | Tempo relativo ao TD (s) e tempo absoluto (s). |
| `heel_y_px`, `toe_y_px` | Altura (px) de calcanhar e antepé (eixo Y da imagem). |
| `knee_deg_raw`, `ankle_deg_raw` | Ângulos brutos (°). |
| `knee_deg_filt`, `ankle_deg_filt` | Ângulos filtrados (°). |
| `hip_x_px`, `hip_y_px` | Posição do quadril (px). |
| `hip_vx_px_s`, `hip_vy_px_s`, `hip_v_px_s` | Velocidade do quadril (px/s). |
| `hip_vx_m_s`, `hip_vy_m_s`, `hip_v_m_s` | Velocidade do quadril (m/s), se houver escala. |
| `hip_KE_J` | Energia cinética (J), se houver massa + escala. |

#### Aba **Norm100**
| Coluna | Significado |
|---|---|
| `name`, `side` | Identificação. |
| `phase_pct` | 0..100% do apoio. |
| `knee_deg_filt`, `ankle_deg_filt` | Séries reamostradas (°). |

#### Aba **Bruto_Marcado**
| Coluna | Significado |
|---|---|
| `frame`, `t_abs_s`, `t_rel_s` | Identificação temporal de cada amostra. |
| `hip_x/y`, `knee_x/y`, `ankle_x/y`, `heel_x/y`, `toe_x/y` | Coordenadas XY (px) no apoio. |

---

# 2) Pronação (vista posterior, marcação manual)

Marque **4 pontos** por frame (**TP, TD, CS, CI**). O script calcula o **ângulo assinado** entre **tíbia** e **calcâneo**, agrupa **contatos** (blocos de frames consecutivos) e gera saídas.

### Saídas
- `Videos/NOME_pronation_overlay_SIDE.mp4` — overlay **apenas onde há continuidade** (gap==step).  
- `Graficos/NOME_pronation_norm100_SIDE.png` — todos os **contatos** normalizados (0–100%).  
- `Dados_Gerais/NOME_PRONATION_SIDE_dados_gerais.xlsx` (um único arquivo com abas):  
  - **Medias** *(primeira aba)* — estatísticas globais  
  - **Discretos** — 1 linha por **contato**  
  - **Continuo** — série empilhada dos contatos  
  - **Norm100** — contatos reamostrados 0–100%  
  - **Bruto_Marcado** — frames marcados (XY + ângulo)

### Dicionário de dados — Pronation

#### Aba **Medias**
| Coluna | Significado |
|---|---|
| `n_contatos` | Número total de contatos (blocos consecutivos). |
| `n_frames_marcados_total` | Total de frames marcados (4 pontos). |
| `step` | Salto de frames usado (~ `round(fps/15)`). |
| `fps`, `meta_rot_deg`, `forced_rot_deg` | Metadados do vídeo/rotação. |
| `n_frames_marcados_mean`, `n_frames_marcados_sd` | Média e DP de frames marcados por contato. |
| `dur_total_s` | Duração total somada dos contatos (s). |
| `dur_media_s`, `dur_dp_s` | Média e DP da duração (s) por contato. |
| `pron_min_deg_mean`, `pron_max_deg_mean`, `pron_ROM_deg_mean` | Médias das métricas discretas (°). |
| `pron_mean_deg_mean`, `pron_sd_deg_mean` | Médias (°) por contato. |
| `pron_peak_deg_mean`, `pron_peak_time_s_mean`, `pron_peak_phase_pct_mean` | Médias dos picos por contato. |
| `pron_at_start_deg_mean`, `pron_at_end_deg_mean` | Médias nas bordas do contato. |

#### Aba **Discretos**
| Coluna | Significado |
|---|---|
| `name`, `side`, `contato_id` | Identificação e índice do contato. |
| `start_frame`, `end_frame` | Limites do contato (frames originais). |
| `start_time_s`, `end_time_s`, `duration_s` | Tempos inicial, final e duração (s). |
| `n_frames_marcados` | Nº de frames com 4 pontos marcados no contato. |
| `pron_min_deg`, `pron_max_deg`, `pron_ROM_deg` | Mín., máx. e amplitude da pronação (°). |
| `pron_mean_deg`, `pron_sd_deg` | Média e DP (°). |
| `pron_peak_deg` | Maior |°| no contato, com **sinal**. |
| `pron_peak_time_s`, `pron_peak_phase_pct` | Tempo absoluto do pico e sua fase % no contato. |
| `pron_at_start_deg`, `pron_at_end_deg` | Ângulos na primeira/última amostra do contato. |
| `fps`, `meta_rot_deg`, `forced_rot_deg` | Metadados. |

#### Aba **Continuo**
| Coluna | Significado |
|---|---|
| `name`, `side`, `contato_id` | Identificação. |
| `frame`, `t(s)` | Índice de frame e tempo absoluto (s). |
| `pronacao_deg` | Sinal +° (pronação), −° (supinação). |

#### Aba **Norm100**
| Coluna | Significado |
|---|---|
| `phase_pct` | 0..100% do contato. |
| `Contato_XX` | Curva ° reamostrada por contato. |

#### Aba **Bruto_Marcado**
| Coluna | Significado |
|---|---|
| `name`, `side`, `contato_id`, `frame`, `t(s)` | Identificação temporal. |
| `tp_x/y`, `td_x/y`, `cs_x/y`, `ci_x/y` | Coordenadas dos 4 pontos (px). |
| `pronacao_deg` | Ângulo calculado no frame marcado (°). |

---

# 3) CMJ / Vertical Jump (sagital, 1 perna)

Consolida **eventos**, **alturas** e **amplitudes** articulares do salto com contramovimento.

### Saídas
- **PNG**: curvas com eventos (CM_start, BOTTOM, TO, LAND).  
- **MP4**: overlay com esqueleto e ângulos.  
- `Dados_Gerais/NOME_CMJ_SIDE_consolidado.xlsx` (um único arquivo com abas):  
  - **Discretos** — identificação, calibração, eventos, alturas, ROMs, ângulos em eventos, meta  
  - **Continuo** — séries temporais filtradas (ângulos; alturas escaladas)

### Dicionário de dados — CMJ (resumo)

#### Aba **Discretos**
| Grupo | Colunas |
|---|---|
| Identificação/escala | `name`, `side`, `thigh_cm_input`, `thigh_px_median`, `knee_filter_used`, `cm_per_px`, `m_per_px` |
| Eventos (s) | `CM_start_time_s`, `BOTTOM_time_s`, `TO_time_s`, `LAND_time_s`, `flight_time_s` |
| Alturas (cm) | `jump_height_by_time_cm`, `jump_height_by_pelvis_cm` |
| Amplitudes (°) | `hip_min_deg/max/ROM`, `knee_min_deg/max/ROM`, `ankle_min_deg/max/ROM` |
| Ângulos em eventos | `*_at_BOTTOM_deg`, `*_at_LAND_deg` |
| Meta | `fps`, `forced_rot_deg` |

#### Aba **Continuo**
| Coluna | Significado |
|---|---|
| `t_s` | Tempo absoluto (s). |
| `hip_deg`, `knee_deg`, `ankle_deg` | Ângulos (°). |
| `pelvis_y_cm`, `heel_y_cm`, `toe_y_cm` | Alturas relativas escaladas (cm). |

---

## Estrutura de pastas (saída)

```
<saida>/
  Videos/
    <arquivos .mp4 com overlay>
  Graficos/
    <arquivos .png com curvas>
  Dados_Gerais/
    <UM .xlsx por análise com todas as abas>
```

---

## Boas práticas e notas
- **Ângulos**: Joelho (flexão +); Tornozelo (0° neutro; dorsi +; planti −); Pronação (posterior) > 0 = pronação.  
- **Qualidade de vídeo**: plano adequado, luz e segmento inteiro visível reduzem falhas.  
- **Falhas de detecção**: verifique **MP4** e **PNGs**; há XLSX(s) com séries de depuração.  
- Encerramento seguro: `cap.release()`, `cv2.destroyAllWindows()`, `plt.close()`; MediaPipe `pose.close()`.  

---

## Licença & citação
Uso acadêmico/interno. Cite este repositório em relatórios/teses quando aplicar as métricas.
