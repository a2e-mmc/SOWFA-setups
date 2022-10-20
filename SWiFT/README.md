# SOWFA-setups for the SWiFT diurnal cycle

SOWFA-setups for the SWiFT diurnal cycle case (8-9 November 2013).

The various subdirectories contain setup files for simulations employing various large-scale forcing methodologies, as summarised in the table below (IPA - indirect profile assimilation; DPA - direct profile assimilation; BCC - budget components coupling)
| Subdirectory | Momentum forcing | Virtual potential temperature forcing |
|--------------|------------------|---------------------------------------|
| internal.ipa.obs.noT | IPA w/ data from tower + radar | no forcing |
| internal.ipa.obs.Tassim | IPA w/ data from tower + radar | forcing at single height |
| internal.ipa.obs.allT | IPA w/ data from tower + radar | IPA w/ data from tower + RASS + sounding |
| internal.dpa.obs.noT | DPA w/ data from tower + radar | no forcing |
| internal.dpa.obs.Tassim | DPA w/ data from tower + radar | forcing at single height |
| internal.dpa.obs.allT | DPA w/ data from tower + radar | DPA w/ data from tower + RASS + sounding |
| internal.ipa.obs.wrfT | IPA w/ data from tower + radar | IPA w/ data from WRF |
| internal.ipa.obs.wrfTadv | IPA w/ data from tower + radar | BCC w/ data from WRF |
| internal.ipa.wrf | IPA w/ data from WRF | IPA w/ data from WRF |
| internal.bcc.wrf | BCC w/ data from WRF | BCC w/ data from WRF |
